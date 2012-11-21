# skでのajax
naoki 2012/11/21

!

## djangoの短所

- ふるき良きRequest and Response
- jsでのレンダリングに向かない
- フルサーバレンダリング

!

## skでのajax

- RESTの設計
- (疑似)透過的API
- knockout経由でのテンプレートバインディング


!

## RESTの設計

- django-rest-framework
	- http://django-rest-framework.org/
- django-piston
	- https://bitbucket.org/jespern/django-piston/wiki/Home
- django-tastypie
	- https://github.com/toastdriven/django-tastypie


!

## RESTの設計

これらは使わず自前実装

- djangoのserializerがバカみたいに遅い
- skのauthはかなりオレオレ
- authとserializeをコントロールしたかった
- のは建前で自分で詳しく知りたかった
- いろいろと設計を変えてみたかった
- REST越しでRPCをやってみたかった

!

## RESTの設計(1)

 - endpoint

```js
'/list/(?P<modelname>\w+)/$'
'/detail/(?P<modelname>\w+)/(?P<modelpk>\d+)/$'
'/create/(?P<modelname>\w+)/$'
'/update/(?P<modelname>\w+)/(?P<modelpk>\d+)/$'
'/delete/(?P<modelname>\w+)/(?P<modelpk>\d+)/$'
```

```js
'/bulk_update/(?P<modelname>\w+)/$'
'/get_or_create/(?P<modelname>\w+)/$'
```

```js
'/custom/(?P<modelname>\w+)/(?P<funcname>\w+)/$'
'/call/(?P<modelname>\w+)/(?P<modelpk>\d+)/(?P<funcname>\w+)/$'
```


!

## RESTの設計(2)

- resource

```python
class EventReservedRecipientResource(Resource):
	# モデルとの連結
    model = EventReservedRecipient
	
	# 展開するかどうか
    modelklss = {
        'recipient': UserProfile,
        'event': Event
    }
	
	# authなどjsレンダリング前の処理
    @classmethod
    def postprocess_query(cls, request, query, modelname, func, resource, temp, option=None):
        if not request.euser:
            raise SKAuthError(request, USER_TYPE_ENTERPRISE)
        temp['sender'] = request.user.up()
        return temp

```

!

## (疑似)透過的API

django本体のORMとほぼ同じように使えるように設計．

```python
Event.objects.filter(name__contains='test')
```

js側のライブラリ

```coffeescript
Event.list {name__contains:'test'}
```

obj-cのライブラリ

```objective-c
Event.sync_filter(@{@"name__contains": @"test"}, nil);
```

node.jsならもっと綺麗に．．meteorとか．

!

## knockout経由でのテンプレートバインディング
- ko.observableArrayを継承したものを返す．

```javascript
events = Event.observableArray([])
events.list {name__contains:'test'}, (val)=>
```

```html
<div data-bind="foreach: events">
	<span data-bind="text: name"></span>
</div>
```

- これでサーバ側フェッチからレンダリングの間のバインディングを全部自動化

!

## 反省点

- SEOで邪魔
- クライアント側でのレンダリングは重い
- クライアント側でのエラー通知
- formと相性が悪い

これらの問題を解決しないと実用性が薄い

!

## 考えられる解決策

- SEOで邪魔
	- 無視(SEO使わないところだけに使う)
	- ヘッドレスブラウザでサーバ側レンダリング？
- クライアント側でのレンダリングは重い
	- 一部サーバレンダリング
	- 再レンダリングを避ける
	- CSSの最適化
- クライアント側でのエラー通知
	- ajax経由で通知
- formと相性が悪い
	- ここは静的にレンダリングすべきか？？
	- validation情報などはサーバ側から送るべき

!

## 連絡事項

! 

## 個別勉強会

- いくつか個別のショートスパンの勉強会をやろうと考えています．
- 意見募集中

- node.js
- ECフレームワーク
- データベース（データマイニング）
