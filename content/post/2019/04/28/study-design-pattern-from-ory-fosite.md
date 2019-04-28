+++
title= "ory/fositeから読み解くGoにおけるStrategy/FactoryMethodパターン"
date= 2019-04-28T17:05:19+09:00
draft = false
toc = true
slug = ""
author = "budougumi0617"
categories = ["go"]
tags = ["golang", "oauth", "fosite", "design"]
keywords = ["Go", "golang", "design pattern", "ory/fosite", "Strategy", "Factory Method"]
twitterImage = "logos/Go-Logo_Aqua.png"
+++

`ory/fosite`というGoのSDKの中でいくつかのデザインパターンが利用されていたので、それを読み解いてみる。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/ory/fosite" data-iframely-url="//cdn.iframe.ly/rg9NVGS"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
<!--more-->

# TL;DR
- `ory/fosite`はOAuth2.0に対応した認可サーバを作るためのGoのSDK
  - https://github.com/ory/fosite
- トークンの払い出しなどの実装はStrategyパターンとFactoryMehotdパターンを利用している
- 上記のデザインパターン以外にも、Functional optionsパターンや型アサーションを組み合わせた、実践的な設計アプローチがなされている
- ソースコードを読んでみることでGoにおけるデザインパターンの実装を学ぶことができた

なお、本記事で参照している`ory/fosite`のコードは2019/04/28時点で最新の`v0.29.6`になる。

- https://github.com/ory/fosite/tree/v0.29.6

# ory/fositeとは
`ory/fosite`はGoで実装されたOAuth2.0のサーバを構築するためのフレームワークだ。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/ory/fosite" data-iframely-url="//cdn.iframe.ly/rg9NVGS"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>
ORY自体はドイツの会社で、他にも`ory/hydra`ライブラリなど、OAuth2.0やOpenID Connectに関するプロダクトを複数提供している。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://www.ory.sh/" data-iframely-url="//cdn.iframe.ly/y8gOpZP"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

GoでOAuth2.0のサーバを構築するライブラリは他にも[`openshift/osin`](https://github.com/openshift/osin)や[`RichardKnop/go-oauth2-server`](https://github.com/RichardKnop/go-oauth2-server)が存在する。
が、`openshift/osin`は既にDuprecatedになっており、Duprecatedを周知するIssueの中で`ory/fosite`が勧められている。

<div class="iframely-embed"><div class="iframely-responsive" style="height: 140px; padding-bottom: 0;"><a href="https://github.com/openshift/osin/issues/186" data-iframely-url="//cdn.iframe.ly/3V8RyPe"></a></div></div><script async src="//cdn.iframe.ly/embed.js" charset="utf-8"></script>

# ory/fositeで活用されているデザインパターン
この記事で触れる`ory/fosite`のコードでは主に次のデザインパターンを利用している。

- FactoryMethodパターン
- Strategyパターン
- Functional optionsパターン

FactroyMethodパターン、Strategyパターンはオーソドックスな[GoFのデザインパターン](https://ja.wikipedia.org/wiki/%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3_(%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2))なので概要や利点の説明は省く。

- 4. FactoryMethod パターン | TECHSCORE
  - https://www.techscore.com/tech/DesignPattern/FactoryMethod.html/
- 10. Strategy パターン | TECHSCORE
  - https://www.techscore.com/tech/DesignPattern/Strategy.html/

## Functional optionsパターン
Functional OptionsパターンはGoで多用されるオプションパターンだ。次のサンプルコードは原典であるRob Pike氏の記事からの引用だ。

- Self-referential functions and the design of options | command center
  - https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html)


```go
type option func(*Foo) interface{}

// Verbosity sets Foo's verbosity level to v.
func Verbosity(v int) option {
    return func(f *Foo) interface{} {
        previous := f.verbosity
        f.verbosity = v
        return previous
    }
}

// Option sets the options specified.
// It returns the previous value of the last argument.
func (f *Foo) Option(opts ...option) (previous interface{}) {
    for _, opt := range opts {
        previous = opt(f)
    }
    return previous
}
```

`func FooOption() option`関数を複数定義し、ビルド関数に可変長引数で`option`型を受けるようにしておくことで、ビルド関数の引数が増えるのを防ぐことが出来る。

# ory/fositeのコードでどのようにデザインパターンが用いられているか
実際に`ory/fosite`でどのようにデザインパターンが利用されているか確認する。

## strategyパターンが使われているory/fositeの基本構造
まず、`ory/fosite`を使うときの基本操作からデザインパターンを見てみる。ここではStrategyパターンによる実処理の移譲が行われている。  
`ory/fosite`の利用者が扱う基本的なインターフェースは[`fosite.OAuth2Provider`](https://github.com/ory/fosite/blob/v0.29.6/oauth2.go#L42-L161)だ。

```go
// https://github.com/ory/fosite/blob/v0.29.6/oauth2.go#L42-L161

// OAuth2Provider is an interface that enables you to write OAuth2 handlers with only a few lines of code.
// Check fosite.Fosite for an implementation of this interface.
type OAuth2Provider interface {
	NewAuthorizeRequest(ctx context.Context, req *http.Request) (AuthorizeRequester, error)

	NewAuthorizeResponse(ctx context.Context, requester AuthorizeRequester, session Session) (AuthorizeResponder, error)

	WriteAuthorizeError(rw http.ResponseWriter, requester AuthorizeRequester, err error)

	WriteAuthorizeResponse(rw http.ResponseWriter, requester AuthorizeRequester, responder AuthorizeResponder)
	
	NewAccessRequest(ctx context.Context, req *http.Request, session Session) (AccessRequester, error)

	// ...
}
```

SDK利用者はHTTPサーバを立ち上げたあと、各HTTPハンドラの中で`fosite.OAuth2Provider`インターフェースを使って認可コードを発行したり、トークンを払い出したりする処理を実装していく（OAuth2.0についての説明は主旨と異なるのでしない）。

このインターフェースの具象型は`fosite.Fosite`型だ。

```go
// https://github.com/ory/fosite/blob/v0.29.6/fosite.go#L85-L105

// Fosite implements OAuth2Provider.
type Fosite struct {
	Store                      Storage
	AuthorizeEndpointHandlers  AuthorizeEndpointHandlers
	TokenEndpointHandlers      TokenEndpointHandlers
	TokenIntrospectionHandlers TokenIntrospectionHandlers
	RevocationHandlers         RevocationHandlers
	Hasher                     Hasher
	ScopeStrategy              ScopeStrategy
	AudienceMatchingStrategy   AudienceMatchingStrategy
	JWKSFetcherStrategy        JWKSFetcherStrategy
	HTTPClient                 *http.Client

	// TokenURL is the the URL of the Authorization Server's Token Endpoint.
	TokenURL string

	// SendDebugMessagesToClients if set to true, includes error debug messages in response payloads. Be aware that sensitive
	// data may be exposed, depending on your implementation of Fosite. Such sensitive data might include database error
	// codes or other information. Proceed with caution!
	SendDebugMessagesToClients bool
}
```

内部に複数の`FooHandlers`を持っている。`FooHandlers`型は`[]FooHandler`に名前付けした型だ。`fosite.Fosite`型は`FooHandlers`に実処理を移譲している。


例えば、`fosite.OAuth2Provider`インターフェースに定義されていて、`fosite.Fosite`で実装されている`NewAccessRequest`メソッドの実装をみてみる。

```go
// https://github.com/ory/fosite/blob/v0.29.6/access_request_handler.go#L87-L101

func (f *Fosite) NewAccessRequest(ctx context.Context, r *http.Request, session Session) (AccessRequester, error) {
	// 前処理
	
	var found bool = false
	for _, loader := range f.TokenEndpointHandlers {
		if err := loader.HandleTokenEndpointRequest(ctx, accessRequest); err == nil {
			found = true
		} else if errors.Cause(err).Error() == ErrUnknownRequest.Error() {
			// do nothing
		} else if err != nil {
			return accessRequest, err
		}
	}

	if !found {
		return nil, errors.WithStack(ErrInvalidRequest)
	}
	return accessRequest, nil
}
```

`fosite.Fosite`型のメソッド内ではロジックの実装は行われていない。
実際の処理は`fosite.Fosite`型の`TokenEndpointHandlers`フィールドで保持している`TokenEndpointHandler`オブジェクトが行なっている。  
`if`文の中の処理を説明すると、いずれかの`TokenEndpointHandler`オブジェクトがリクエストを処理できれば`NewAccessRequest`メソッドは正常終了する。
`TokenEndpointHandler`オブジェクトが`ErrUnknownRequest`エラーを返した場合、それはその`TokenEndpointHandler`オブジェクトでは未サポートのリクエストだったことになる。いずれかの`TokenEndpointHandler`オブジェクトが`ErrUnknownRequest`以外のエラーを返した場合、、あるいは全ての`TokenEndpointHandler`オブジェクトが処理できなかった場合は異常なリクエストだったとしてエラーを返して終わる。
  
では、次に`fosite.Fosite`の`FooHandlers`フィールドに実処理を行なう各`Handler`を格納する実装を見てみる。

## Functional Optionsパターンを利用したStrategyパターンの初期化とFactoryメソッドパターン
`fosite.Fosite`オブジェクトのフィールドは外部に公開されている。そのため、地道に各オブジェクトを設定していくこともできるが、`ory/fosite`では初期化用の`compose.Compose`関数が用意されている。

```go
https://github.com/ory/fosite/blob/v0.29.6/compose/compose.go#L33-L91

type Factory func(config *Config, storage interface{}, strategy interface{}) interface{}


// Compose takes a config, a storage, a strategy and handlers to instantiate an OAuth2Provider:
// ...
func Compose(config *Config, storage interface{}, strategy interface{}, hasher fosite.Hasher, factories ...Factory) fosite.OAuth2Provider {
	if hasher == nil {
		hasher = &fosite.BCrypt{WorkFactor: config.GetHashCost()}
	}

	f := &fosite.Fosite{
		Store:                      storage.(fosite.Storage),
		AuthorizeEndpointHandlers:  fosite.AuthorizeEndpointHandlers{},
		TokenEndpointHandlers:      fosite.TokenEndpointHandlers{},
		TokenIntrospectionHandlers: fosite.TokenIntrospectionHandlers{},
		RevocationHandlers:         fosite.RevocationHandlers{},
		Hasher:                     hasher,
		ScopeStrategy:              config.GetScopeStrategy(),
		AudienceMatchingStrategy:   config.GetAudienceStrategy(),
		SendDebugMessagesToClients: config.SendDebugMessagesToClients,
		TokenURL:                   config.TokenURL,
		JWKSFetcherStrategy:        config.GetJWKSFetcherStrategy(),
	}

	for _, factory := range factories {
		res := factory(config, storage, strategy)
		if ah, ok := res.(fosite.AuthorizeEndpointHandler); ok {
			f.AuthorizeEndpointHandlers.Append(ah)
		}
		if th, ok := res.(fosite.TokenEndpointHandler); ok {
			f.TokenEndpointHandlers.Append(th)
		}
		if tv, ok := res.(fosite.TokenIntrospector); ok {
			f.TokenIntrospectionHandlers.Append(tv)
		}
		if rh, ok := res.(fosite.RevocationHandler); ok {
			f.RevocationHandlers.Append(rh)
		}
	}

	return f
}
```

`compose.Compose`関数はFunctional OptionsパターンとFactoryMethodパターンを利用している。  
引数で受けた可変長引数の`factories`は後述する`FooHnadler`オブジェクトを初期化する`FooFactory`関数が満たしている`Factory`型の関数定義だ。
Goはダックタイピングなので、`Factory`が生成した`FooHandler`オブジェクトの実体を知らずとも利用できる。型アサーションで`factory`関数から生成されたオブジェクトがどのインターフェースを満たしているか確認し、インターフェースを満たしていれば、`fosite.Fosite`オブジェクトに登録していく。

`factory`型を満たすファクトリーメソッド自体は、以下のような関数群が定義されている。

```go
// https://github.com/ory/fosite/blob/v0.29.6/compose/compose_oauth2.go

// OAuth2ClientCredentialsGrantFactory creates an OAuth2 client credentials grant handler and registers
// an access token, refresh token and authorize code validator.
func OAuth2ClientCredentialsGrantFactory(config *Config, storage interface{}, strategy interface{}) interface{} {
	return &oauth2.ClientCredentialsGrantHandler{
		HandleHelper: &oauth2.HandleHelper{
			AccessTokenStrategy: strategy.(oauth2.AccessTokenStrategy),
			AccessTokenStorage:  storage.(oauth2.AccessTokenStorage),
			AccessTokenLifespan: config.GetAccessTokenLifespan(),
		},
		ScopeStrategy:            config.GetScopeStrategy(),
		AudienceMatchingStrategy: config.GetAudienceStrategy(),
	}
}
```

```go
// https://github.com/ory/fosite/blob/v0.29.6/compose/compose_openid.go

// OpenIDConnectRefreshFactory creates a handler for refreshing openid connect tokens.
//
// **Important note:** You must add this handler *after* you have added an OAuth2 authorize code handler!
func OpenIDConnectRefreshFactory(config *Config, storage interface{}, strategy interface{}) interface{} {
	return &openid.OpenIDConnectRefreshHandler{
		IDTokenHandleHelper: &openid.IDTokenHandleHelper{
			IDTokenStrategy: strategy.(openid.OpenIDConnectTokenStrategy),
		},
	}
}
```

`ory/fosite`ユーザは自分が利用したい認証・認可形式のファクトリーメソッドを使って`compose.Compose`関数を呼び出すことで、任意の実装形式が設定された`fosite.Oauth2Providor`オブジェクトを取得することができる。

以下は提供されている全てのFactoryを使って`compose.Compose`関数を呼び出して`fosite.OAuth2Provider`の初期化を行なう`compose.ComposeAllEnabled`の定義だ。

```go
// https://github.com/ory/fosite/blob/v0.29.6/compose/compose.go#L93-L122

// ComposeAllEnabled returns a fosite instance with all OAuth2 and OpenID Connect handlers enabled.
func ComposeAllEnabled(config *Config, storage interface{}, secret []byte, key *rsa.PrivateKey) fosite.OAuth2Provider {
	return Compose(
		config,
		storage,
		&CommonStrategy{
			CoreStrategy:               NewOAuth2HMACStrategy(config, secret, nil),
			OpenIDConnectTokenStrategy: NewOpenIDConnectStrategy(config, key),
			JWTStrategy: &jwt.RS256JWTStrategy{
				PrivateKey: key,
			},
		},
		nil,

		OAuth2AuthorizeExplicitFactory,
		OAuth2AuthorizeImplicitFactory,
		OAuth2ClientCredentialsGrantFactory,
		OAuth2RefreshTokenGrantFactory,
		OAuth2ResourceOwnerPasswordCredentialsFactory,

		OpenIDConnectExplicitFactory,
		OpenIDConnectImplicitFactory,
		OpenIDConnectHybridFactory,
		OpenIDConnectRefreshFactory,

		OAuth2TokenIntrospectionFactory,

		OAuth2PKCEFactory,
	)
}
```

# まとめ
GoのOSS SDKどのようにデザインパターンが活用されているかコードリーディングした。  
デザインパターンの紹介記事はよくあるが、自動車とタイヤの簡単なコードだったりで実際のプロダクトのコードに適用するときとはだいぶ乖離がある。それなりのコード規模のSDKの中でどのように使われているか読むことで、自分の設計スキルに大きくプラスになった。  
また、古典的なGoFのデザインパターンと、Goの言語仕様やGo独特（？）のデザインパターンを組み合わせたハイブリットな設計パターンだったのも良かった。
ただ、Goはダックタイピングなので、ある`Handler`型がどのインターフェースを満たしているのかパット見わからない。
実際にはどれが呼び出されているのか？を確認するのは少し時間がかかった。言語仕様上は具象型とインターフェースは疎結合でも、実体は密結合な利用方法をとるので、コメントに実装インターフェースを記載しておくなどの配慮が必要そうだ。

# 参考
- ory/fosite
  - https://github.com/ory/fosite
- ORY
  - https://www.ory.sh/
- #186 Deprecating the project | openshift/osin
  - https://github.com/openshift/osin/issues/186#issuecomment-422597936
- 4. FactoryMethod パターン | TECHSCORE
  - https://www.techscore.com/tech/DesignPattern/FactoryMethod.html/
- 10. Strategy パターン | TECHSCORE
  - https://www.techscore.com/tech/DesignPattern/Strategy.html/
- Self-referential functions and the design of options | command center
  - https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
- Functional options for friendly APIs | Dave Cheney
  - https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis
