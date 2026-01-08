# Laravel로 Webスクレイピング하기

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 Laravel를 사용하여 Webスクレイピング을 수행하는 방법을 설명합니다:

- [Prerequisites](#prerequisites)
- [How to Build a Web Scraping API in Laravel](#how-to-build-a-web-scraping-api-in-laravel)
  - [Step 1: Set up a Laravel Project](#step-1-set-up-a-laravel-project)
  - [Step 2: Initialize Your Scraping API](#step-2-initialize-your-scraping-api)
  - [Step 3: Install the Scraping Libraries](#step-3-install-the-scraping-libraries)
  - [Step 4: Download the Target Page](#step-4-download-the-target-page)
  - [Step 5: Inspect the Page Content](#step-5-inspect-the-page-content)
  - [Step 6: Prepare for Web Scraping](#step-6-prepare-for-web-scraping)
  - [Step 7: Implement Data Scraping](#step-7-implement-data-scraping)
  - [Step 8: Return the Scraped Data](#step-8-return-the-scraped-data)
  - [Step 9: Put It All Together](#step-9-put-it-all-together)
- [Next Steps](#next-steps)

## Laravel에서 Webスクレイピング이 가능한가요?

[Laravel](https://laravel.com/)은 우아한 문법을 갖춘 강력한 PHP 프레임워크로, Webスクレイピング을 위한 API를 구축하는 데 이상적입니다. 다양한 スクレイピング 라이브러리를 지원하여 데이터 추출을 단순화합니다. 

Laravel의 확장성, 쉬운 통합, 강력한 MVC 아키텍처는 スクレイピング 로직을 잘 구조화된 상태로 유지해 주므로, 복잡하거나 대규모 프로젝트에 적합합니다. 자세한 내용은 [PHP에서 web scraping](https://brightdata.co.kr/blog/how-tos/web-scraping-php) 가이드를 참조하십시오.

## 최고의 Laravel Webスクレイピング 라이브러리

다음은 Laravel에서 Webスクレイピング에 유용한 주요 라이브러리입니다:

- [**BrowserKit**](https://github.com/symfony/browser-kit) – 정적 HTML 문서와 상호작용하기 위한 웹 브라우저 API를 시뮬레이션하는 Symfony 컴포넌트입니다. `DomCrawler`와 함께 사용하면 효율적으로 탐색 및 スクレイピング을 수행할 수 있습니다.
- [**HttpClient**](https://github.com/symfony/http-client) – `BrowserKit`과 매끄럽게 통합되어 リクエスト를 전송할 수 있는 Symfony HTTP 클라이언트입니다.
- [**Guzzle**](https://github.com/guzzle/guzzle) – 웹 リクエスト를 생성하고 レスポンス를 처리하기 위한 강력한 HTTP 클라이언트입니다. HTML 문서를 가져오는 데 유용합니다. [Guzzle에서 プロキシ 설정 방법](https://brightdata.co.kr/blog/how-tos/proxy-with-guzzle)을 확인하십시오.
- [**Panther**](https://github.com/symfony/panther) – JavaScript 렌더링 또는 상호작용이 필요한 동적 사이트를 スクレイピング하기 위한 헤드리스 브라우저입니다.
## Prerequisites

Laravel에서 Webスクレイピング 튜토리얼을 따라 하려면 다음 사전 요구 사항을 충족해야 합니다:

- [PHP 8+](https://www.php.net/downloads.php)
- [Composer](https://getcomposer.org/download/)

또한 PHP로 코딩할 수 있는 IDE 사용을 권장합니다.

## How to Build a Web Scraping API in Laravel

이 섹션에서는 [Quotes scraping sandbox site](https://quotes.toscrape.com/)를 사용하여 Laravel Webスクレイピング API를 만드는 과정을 안내합니다. スクレイピング エンドポイント는 다음을 수행합니다:

1. 페이지에서 인용구 quote HTML 요소를 선택합니다  
2. 해당 요소들에서 데이터를 추출합니다  
3. スクレイ핑한 데이터를 JSON 형식으로 반환합니다  

타깃 사이트는 다음과 같이 보입니다:

![Quotes to scrape page](https://github.com/luminati-io/laravel-web-scraping/blob/main/images/68747470733a2f2f627269676874646174612e636f6d2f77702d636f6e74656e742f75706c6f6164732f323032322f31312f51756f7465732d746f2d5363726170652d706167652d6769662e676966.gif)

**Step 1: Set up a Laravel project**

터미널을 열고 아래 Composer [`create-command`](https://getcomposer.org/doc/03-cli.md#create-project) 명령을 실행하여 Laravel Webスクレイピング 애플리케이션을 초기화하십시오:

```bash
composer create-project laravel/laravel laravel-scraper
```

이제 `lavaral-scraper` 폴더에 빈 Laravel 프로젝트가 포함됩니다. 선호하는 PHP IDE로 로드하십시오.

현재 백엔드의 파일 구조는 다음과 같습니다:

![file structure in the backend](https://github.com/luminati-io/laravel-web-scraping/blob/main/images/687474~1.PNG)

**Step 2: Initialize Your Scraping API**

프로젝트 디렉터리에서 아래 [Artisan command](https://laravel.com/docs/11.x/artisan)를 실행하여 새로운 Laravel 컨트롤러를 추가하십시오:

```bash
php artisan make:controller HelloWorldController
```

그러면 `/app/Http/Controllers` 디렉터리에 다음 `ScrapingController.php` 파일이 생성됩니다:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ScrapingController extends Controller

{

//

}
```

`ScrapingController` 파일에 다음 `scrapeQuotes()` 메서드를 추가하십시오:

```
public function scrapeQuotes(): JsonResponse

{

// scraping logic...

return response()->json('Hello, World!');

}
```

현재 이 메서드는 플레이스홀더인 `'Hello, World!'` JSON 메시지를 반환합니다.

다음 import를 추가하십시오:

```php
use Illuminate\Http\JsonResponse;
```

`routes/api.php`에 다음 줄을 추가하여 `scrapeQuotes()` 메서드를 전용 エンドポイント에 연결하십시오:

```php
use App\Http\Controllers\ScrapingController;

Route::get('/v1/scraping/scrape-quotes', [ScrapingController::class, 'scrapeQuotes']);
```

Laravel スクレイピング API가 예상대로 작동하는지 확인해 보겠습니다. Laravel API는 `/api` 경로 아래에서 사용 가능하므로, 전체 API エンドポイント는 `/api/v1/scraping/scrape-quotes`입니다.

Laravel 애플리케이션을 실행하십시오:

```php
php artisan serve
```

이제 서버가 로컬에서 `8000` 포트로 수신 대기 중이어야 합니다.

cURL로 `/api/v1/scraping/scrape-quotes` エンドポイント에 `GET` リクエスト를 보내십시오:

```bash
curl -X GET 'http://localhost:8000/api/v1/scraping/scrape-quotes'
```

다음 レスポンス가 반환되어야 합니다:

```
"Hello, World!"
```

**Step 3: Install the scraping libraries**

패키지를 설치하기 전에, 요구 사항에 맞는 Laravel Webスクレイピング 라이브러리를 결정하십시오. 타깃 사이트를 열고 Developer Tools로 검사한 다음 **Network → Fetch/XHR** 섹션을 확인하십시오:

![Accessing the 'Fetch XHR' section](https://github.com/luminati-io/laravel-web-scraping/blob/main/images/687474~2.PNG)

이 사이트는 [AJAX requests](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)를 수행하지 않으므로, HTML에 데이터가 내장된 정적 페이지입니다. [headless browser](https://brightdata.co.kr/blog/web-data/best-headless-browsers)는 오버헤드를 추가하므로 필요하지 않습니다.

효율적인 スクレイピング을 위해 Symfony의 [`BrowserKit`](https://symfony.com/doc/current/http_client.html)과 [`HttpClient`](https://symfony.com/doc/current/http_client.html)를 사용하십시오. 다음으로 설치합니다:

```bash
composer require symfony/browser-kit symfony/http-client
```

**Step 4: Download the target page**

`ScrapingController`에서 `BrowserKit`과 `HttpClient`를 import하십시오:

```php
use Symfony\Component\BrowserKit\HttpBrowser;

use Symfony\Component\HttpClient\HttpClient;
```

`scrapeQuotes()`에서 새 `HttpBrowser` 객체를 초기화하십시오:

```php
$browser = new HttpBrowser(HttpClient::create());
```

`HttpBrowser`는 Cookie 및 セッション 처리 등을 포함하여 브라우저 동작을 모방하면서 HTTP リクエスト를 수행할 수 있게 해줍니다. 다만 실제 브라우저에서 リクエスト를 실행하지는 않습니다.

`request()` 메서드를 사용하여 타깃 URL로 HTTP GET リクエスト를 수행하십시오:

```php
$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');
```

결과는 서버가 반환한 HTML 문서를 자동으로 파싱하는 [`Crawler`](https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/DomCrawler/Crawler.php) 객체가 됩니다. 이 클래스는 노드 선택 및 데이터 추출 기능도 제공합니다.

crawler에서 페이지의 HTML을 추출하여 위 로직이 동작하는지 확인할 수 있습니다:

```
$html = $crawler->outerHtml();
```

테스트를 위해 API가 이 데이터를 반환하도록 하십시오.

이제 `scrapeQuotes()` 함수는 다음과 같아집니다:

```php
public function scrapeQuotes(): JsonResponse

{

// initialize a browser-like HTTP client

$browser = new HttpBrowser(HttpClient::create());

// download and parse the HTML of the target page

$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');

// get the page outer HTML and return it

$html = $crawler->outerHtml();

return response()->json($html);

}
```

이제 API는 다음을 반환합니다:

```html
<!DOCTYPE html>

<html lang="en">

<head>

<meta charset="UTF-8">

<title>Quotes to Scrape</title>

<link rel="stylesheet" href="/static/bootstrap.min.css">

<link rel="stylesheet" href="/static/main.css">

</head>

<!-- omitted for brevity ... -->
```

**Step 5: Inspect the page content**

추출 로직을 정의하기 위해 타깃 페이지의 HTML 구조를 검사하십시오.

1. [Quotes To Scrape](https://quotes.toscrape.com/)를 여십시오.  
2. 인용구 요소를 우클릭하고 DevTools에서 **Inspect**를 선택하십시오.  
3. HTML을 확장하고 구조를 살펴보십시오:

![Inspecting the quote elements](https://github.com/luminati-io/laravel-web-scraping/blob/main/images/687474~3.PNG)

각 `.quote` 요소는 다음을 포함합니다:  
- 인용구 텍스트를 위한 `.text` 노드  
- 저자 이름을 위한 `.author` 노드  
- 관련 태그를 위한 여러 `.tag` 노드  

이러한 [CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)를 사용하여 Laravel에서 원하는 데이터를 추출할 수 있습니다.

**Step 6: Get ready to perform web scraping**

스크レ이ピング한 데이터를 저장할 데이터 구조를 만드십시오. 이를 위해 배열을 사용합니다:

```php
quotes = []
```

이제 `Crawler` 클래스의 `filter()` 메서드를 사용하여 모든 quote 요소를 선택하십시오:

```php
$quote_html_elements = $crawler->filter('.quote');
```

이는 지정된 `.quote` CSS selector와 일치하는 페이지의 모든 DOM 노드를 반환합니다.

다음으로, 이를 반복하면서 각 요소에 데이터 추출 로직을 적용할 준비를 하십시오:

```php
foreach ($quote_html_elements as $quote_html_element) {

// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// scraping logic...

}
```

`filter()`가 반환하는 `DOMNode` 객체에는 노드 선택 메서드가 없습니다. 이를 해결하기 위해, 특정 HTML quote 요소 범위로 스코프가 제한된 로컬 `Crawler` 인스턴스를 생성합니다.

코드가 올바르게 동작하도록 다음 import를 추가하십시오:

```php
use Symfony\Component\DomCrawler\Crawler;
```

**Step 7: Implement data scraping**

`foreach` 루프 내부에서 다음을 수행합니다:

1. `.text`, `.author`, `.tag` 요소에서 관심 데이터를 추출합니다
2. 이를 사용해 새 `$quote` 객체를 채웁니다
3. 새 `$quote` 객체를 `$quotes`에 추가합니다

먼저 HTML quote 요소 내부의 .text 요소를 선택합니다. 그런 다음 `text()` 메서드를 사용하여 내부 텍스트를 추출합니다:

```php
$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();
```

각 인용구는 `\u201c` 및 `\u201d` 특수 문자로 둘러싸여 있습니다. 다음과 같이 PHP 함수 [`str_replace()`](https://www.php.net/manual/en/function.str-replace.php)를 사용하여 제거할 수 있습니다:

```php
$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);
```

마찬가지로 다음 코드로 저자 정보를 スクレイピング하십시오:

```php
$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();
```

태그 スクレイピング은 까다로울 수 있습니다. 하나의 인용구에 여러 태그가 있을 수 있으므로, 배열을 정의하고 각 태그를 개별적으로 スクレイピング해야 합니다:

```php
$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}
```

`filter()`가 반환하는 `DOMNode` 요소는 `text()` 메서드를 노출하지 않습니다. 대신 `textContent` 속성을 제공합니다.

다음은 전체 Laravel 데이터 スクレイピング 로직입니다:

```php
// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// perform the data extraction logic

$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();

// remove special characters from the raw text information

$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);

$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();

$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}
```

**Step 8: Return the scraped data**

스크レイピング한 데이터로 `$quote` 객체를 만들고 `$quotes`에 추가하십시오:

```php
$quote = [

'text' => $text,

'author' => $author,

'tags' => $tags

];

$quotes[] = $quote;
```

다음으로, API レスポンス 데이터를 `$quotes` 목록으로 업데이트하십시오:

```php
return response()->json(['quotes' => $quotes]);
```

スクレイピング 루프가 끝나면 `$quotes`에는 다음이 포함됩니다:

```php
array(10) {

[0]=>

array(3) {

["text"]=>

string(113) "The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking."

["author"]=>

string(15) "Albert Einstein"

["tags"]=>

array(4) {

[0]=>

string(6) "change"

[1]=>

string(13) "deep-thoughts"

[2]=>

string(8) "thinking"

[3]=>

string(5) "world"

}

}

// omitted for brevity...

[9]=>

array(3) {

["text"]=>

string(48) "A day without sunshine is like, you know, night."

["author"]=>

string(12) "Steve Martin"

["tags"]=>

array(3) {

[0]=>

string(5) "humor"

[1]=>

string(7) "obvious"

[2]=>

string(6) "simile"

}

}

}
```

이 데이터는 이후 JSON으로 직렬화되어 Laravel スクレイピング API에 의해 반환됩니다.

**Step 9: Put it all together**

다음은 Laravel의 `ScrapingController` 파일 최종 코드입니다:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use Illuminate\Http\JsonResponse;

use Symfony\Component\BrowserKit\HttpBrowser;

use Symfony\Component\HttpClient\HttpClient;

use Symfony\Component\DomCrawler\Crawler;

class ScrapingController extends Controller

{

public function scrapeQuotes(): JsonResponse

{

// initialize a browser-like HTTP client

$browser = new HttpBrowser(HttpClient::create());

// download and parse the HTML of the target page

$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');

// where to store the scraped data

$quotes = [];

// select all quote HTML elements on the page

$quote_html_elements = $crawler->filter('.quote');

// iterate over each quote HTML element and apply

// the scraping logic

foreach ($quote_html_elements as $quote_html_element) {

// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// perform the data extraction logic

$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();

// remove special characters from the raw text information

$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);

$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();

$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}

// create a new quote object

// with the scraped data

$quote = [

'text' => $text,

'author' => $author,

'tags' => $tags

];

// add the quote object to the quotes array

$quotes[] = $quote;

}

var_dump($quotes);

return response()->json(['quotes' => $quotes]);

}

}
```

테스트해 보겠습니다. Laravel 서버를 시작하십시오:

```bash
php artisan serve
```

`/api/v1/scraping/scrape-quotes` エンドポイント로 GET リクエスト를 보내십시오:

```bash
curl -X GET 'http://localhost:8000/api/v1/scraping/scrape-quotes'
```

다음 결과를 얻게 됩니다:

```
{

"quotes": [

{

"text": "The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.",

"author": "Albert Einstein",

"tags": [

"change",

"deep-thoughts",

"thinking",

"world"

]

},

// omitted for brevity...

{

"text": "A day without sunshine is like, you know, night.",

"author": "Steve Martin",

"tags": [

"humor",

"obvious",

"simile"

]

}

]

}
```

## Next steps

이 API는 Laravel의 Webスクレイピング 기능에 대한 기본 예시입니다. 프로젝트를 개선하고 확장하려면 다음과 같은 향상을 고려하십시오:

- **Webクローリング 구현** – 타깃 사이트는 여러 페이지로 구성되어 있습니다. Webクローリング을 사용하여 모든 인용구를 효율적으로 수집하십시오.  
- **スクレイピング 작업 스케줄링** – API 호출을 스케줄링하고, 데이터를 데이터베이스에 저장하며, 최신 상태를 유지하여 데이터 수집을 자동화하십시오.  
- **プロキシ 통합** – レジデンシャルプロキシ를 사용해 リクエスト를 분산하고 アンチボット/アンチスクレイピング 조치를 우회하여 IP 차단을 피하십시오.  

## 윤리적 Webスクレイピング

Webスクレイピング은 데이터 수집을 위한 강력한 도구이지만, 윤리적이고 책임감 있게 수행되어야 합니다. 준수를 보장하고 타깃 사이트에 피해를 주지 않기 위해 다음 모범 사례를 따르십시오:

- **사이트의 서비스 약관 검토** – スクレイピング 전에 데이터 사용, 저작권, 지적 재산권에 관한 가이드라인을 확인하십시오.  
- **`robots.txt` 규칙 준수** – 윤리적인 スクレイピング 관행을 유지하기 위해 사이트의 クローリング 지침을 따르십시오.  
- **공개 데이터만 スクレイ핑** – 認証이 필요한 제한된 콘텐츠는 피하십시오. 비공개 데이터를 スクレイピング하면 법적 결과가 발생할 수 있습니다.  
- **リクエスト 빈도 제한** – リクエスト 간격을 조절하고 무작위 지연을 추가하여 서버 과부하와 レート制限을 방지하십시오.  
- **신뢰할 수 있는 スクレイピング 도구 사용** – 윤리적 スクレイピング 가이드라인을 따르는, 유지보수가 잘 되는 도구를 선택하십시오.

## 결론

Laravel로 Webスクレイピング을 수행하는 것은 간단하며 몇 줄의 코드만으로도 가능합니다. 그러나 대부분의 사이트는 アンチボット 및 アンチスクレイピング 솔루션으로 데이터를 보호합니다. 이를 우회하려면, 어떤 페이지든 アンチスクレイピング 조치를 우회하여 깨끗한 HTML을 매끄럽게 반환할 수 있는 언락킹 API인 [**Web Unlocker**](https://brightdata.co.kr/products/web-unlocker)를 사용할 수 있습니다.

지금 가입하고 무료 체험을 시작하십시오.