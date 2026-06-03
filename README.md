# MoeImgTag

为高版本 MediaWiki 添加 `<img>` 标签支持。包含防 xss 攻击的过滤。

## 安装

```php
wfLoadExtension( 'MoeImgTag' );
```

## 使用方法

### 扩展标签

```html
<!-- 最小可用的示例 -->
<img src="https://example.com/image.jpg" />

<!-- 更多可用的属性 -->
<img
  src="https://example.com/image.jpg"
  class="my-class"
  style="width: 100%; height: auto;"
  loading="lazy"
  referrerpolicy="no-referrer"
  title="My Image"
  alt="My Image"
  width="400"
  height="300"
/>
```

> [!IMPORTANT]
> img 标签必须被正确闭合：`/>`

这种用法可以简单的在文章内嵌入行内外部图片。

但需注意，任何参数都会被当做纯文本解析，即你无法在参数中使用解析器函数或模板调用。如有需要，请使用解析器函数。

### 解析器函数

```plaintext
{{ #img: https://example.com/image.jpg }}
```

> 该函数可以认为是 `{{ #tag: img | src = https://example.com/image.jpg }}` 的语法糖。

你可以在参数中使用解析器函数或模板调用，例如：

```plaintext
{{ #img: {{ filepath: {{{1}}} }}
  | class = my-class {{{className|}}}
  | style = width: 100%; height: auto; {{{style|}}}
  | loading = {{{loading|lazy}}}
  | referrerpolicy = {{{referrerpolicy|}}}
  | title = {{{title|}}}
  | alt = {{{alt|}}}
  | width = {{{width|}}}
  | height = {{{height|}}}
}}
```

## 配置项

### 黑名单&白名单

```php
$wgMoeImgTagWhitelist = [
    'example.com',
    '*.example.com',
    'https://example.com',
    'https://*.example.com',
    'https://example.com/only/allow/this/path',
];
$wgMoeImgTagBlacklist = [
    // format is the same as whitelist
];
```

白名单和黑名单互相排斥，如果同时设置，则只会使用白名单。

匹配的规则可以参考 [Renderer::checkUrlMatch](./includes/Renderer.php) 的实现。

> [!TIP]
> 设置白名单的时候，别忘了把本站域名加上，最简单的方式是：

```php
$wgMoeImgTagWhitelist = [
    $wgServer,
    // ...
];
```

### 默认空 Referer 域名

```php
$wgMoeImgTagNoReferrerDomains = [
    'hdslb.com',
];
```

若图片的 `src` 命中这些域名，且调用方没有显式设置 `referrerpolicy`，扩展会自动输出 `referrerpolicy="no-referrer"`。

域名会按后缀匹配，因此 `hdslb.com` 同时匹配 `hdslb.com` 和 `i0.hdslb.com`，但不会匹配 `evil-hdslb.com`。

### 修复自闭合标签 (实验性)

```php
$wgMoeImgTagFixSelfClosing = true;
```

若开启此选项，扩展会尝试在解析文本前修复未闭合的 `<img>` 标签。

例如：`<img src="https://example.com/image.jpg">` 会被转换为 `<img src="https://example.com/image.jpg" />`。

虽然 HTML5 规范中 `<img>` 标签是可以自闭合的，但 wikitext 解析器并不认识这种写法，因此我们提供了这种曲线救国的方式。

有可能会导致一些意外的结果，因此默认情况下不开启。
