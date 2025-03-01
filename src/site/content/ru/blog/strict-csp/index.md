---
layout: post
title: Предотвращение межсайтового скриптинга (XSS) при помощи строгой политики безопасности контента (CSP)
subhead: Развертывание CSP-политики на основе одноразовых номеров или хешей скриптов для обеспечения комплексной защиты от межсайтового скриптинга.
description: Узнайте, как развернуть CSP-политику на основе одноразовых номеров или хешей скриптов, чтобы обеспечить комплексную защиту от межсайтового скриптинга.
authors:
  - lwe
date: 2021-03-15
hero: image/3lmWcR1VGYVMicNlBh4aZWBTcSg1/mhE0NYvP3JFyvNyiQ1dj.jpg
alt: Скриншот JavaScript-кода, устанавливающего строгую политику безопасности контента.
tags:
  - blog
  - security
---

## Зачем нужна строгая политика безопасности контента (CSP)?

[Межсайтовый скриптинг (XSS)](https://www.google.com/about/appsecurity/learning/xss/) — внедрение в веб-приложения вредоносных скриптов — уже более десяти лет является одной из самых серьезных уязвимостей в области веб-безопасности.

[Политика безопасности контента (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) — это дополнительная мера защиты, нацеленная на предотвращение XSS-атак. Для настройки CSP-политики необходимо добавить на веб-страницу HTTP-заголовок Content-Security-Policy и установить значения, чтобы указать, какие ресурсы пользовательскому агенту разрешено загружать в рамках этой страницы. Эта статья рассказывает, как защититься от XSS-атак, используя CSP-политику на основе одноразовых номеров или хешей в противоположность стандартным CSP-политикам на основе белого списка доменов, которые зачастую оставляют страницу уязвимой для XSS-атак, поскольку в большинстве конфигураций [их можно обойти](https://research.google.com/pubs/pub45542.html).

{% Aside 'key-term' %} *Одноразовый номер* — это случайное, используемое только один раз число, при помощи которого можно пометить тег `<script>` как доверенный. {% endAside %}

{% Aside 'key-term' %} Хеш-функция — это математическая функция, преобразующая подаваемое на вход значение в сжатое число — хеш. При помощи *хешей* (таких, как [SHA-256](https://en.wikipedia.org/wiki/SHA-2)) можно помечать встраиваемые теги `<script>` как доверенные. {% endAside %}

Политика безопасности контента, основанная на использовании одноразовых номеров или хешей, часто называется *строгой CSP-политикой*. Если приложение использует строгую CSP-политику, злоумышленники, обнаруживающие неправильно внедренный HTML-код, в большинстве случаев не смогут с его помощью заставить браузер выполнять вредоносные скрипты в контексте уязвимого документа. Это обусловлено тем, что при использовании строгой CSP-политики разрешается выполнение только хешированных скриптов или скриптов с верно указанным одноразовым номером, сгенерированным на сервере, благодаря чему злоумышленник не может выполнить скрипт, так как не имеет одноразового номера, соответствующего ответу.

{% Aside %} Чтобы защитить сайт от XSS-атак, необходимо выполнять санитизацию вводимых пользователем данных и *одновременно* использовать CSP-политику в качестве дополнительной меры защиты. CSP — это метод [комплексной защиты](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)), который может предотвратить выполнение вредоносных скриптов, однако он не избавляет от необходимости предотвращать (и вовремя исправлять) ошибки XSS. {% endAside %}

### Почему строгая CSP-политика предпочтительнее CSP-политики на основе белого списка

Если на вашем сайте применяется CSP-политика с явным указанием домена (`script-src www.googleapis.com`), имейте в виду, что она может не быть эффективным средством защиты от межсайтового скриптинга. Такая CSP-политика называется политикой на основе белого списка, и у нее есть два недостатка:

- Она требует существенного изменения кода.
- В большинстве вариантов конфигурации [ее можно обойти](https://research.google.com/pubs/pub45542.html).

Эти недостатки делают CSP на основе белого списка неэффективным средством защиты от XSS-атак. Вот почему рекомендуется использовать строгую CSP-политику на основе криптографически сгенерированных одноразовых номеров или хешей, чтобы избежать проблем, описанных выше.

<div class="switcher">{% Compare 'worse', 'CSP-политика на основе белого списка' %} — Не является эффективным средством защиты сайта. ❌ — Требует существенного изменения кода. 😓 {% endCompare %}</div>
<p data-md-type="paragraph">{% Compare 'better', 'Строгая CSP-политика' %}</p>
<ul data-md-type="list" data-md-list-type="unordered" data-md-list-tight="true">
<li data-md-type="list_item" data-md-list-type="unordered">Эффективно защищает ваш сайт. ✅</li>
<li data-md-type="list_item" data-md-list-type="unordered">Код всегда имеет одинаковую структуру. 😌 {% endCompare %}</li>
</ul>
<div data-md-type="block_html"></div>

## В чем заключается суть строгой политики безопасности контента?

Строгая политика безопасности контента настраивается в соответствии со структурой, показанной ниже, и для ее активации необходимо установить один из следующих заголовков ответа HTTP:

- **Строгая CSP-политика на основе одноразовых номеров**

```text
Content-Security-Policy:
  script-src 'nonce-{RANDOM}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';

```

{% Img src="image/vgdbNJBYHma2o62ZqYmcnkq3j0o1/er4BaGCJzBwDaESFKfZd.jpg", alt="", width="800", height="279" %}

- **Строгая CSP-политика на основе хешей**

```text
Content-Security-Policy:
  script-src 'sha256-{HASHED_INLINE_SCRIPT}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';

```

{% Aside warning %} Здесь показана минимальная версия строгой CSP-политики. Чтобы обеспечить работу во всех браузерах, ее необходимо дополнить; подробности см. в разделе об [указании резервной политики для поддержки Safari и устаревших браузеров](#step-4:-add-fallbacks-to-support-safari-and-older-browsers). {% endAside %}

CSP-политика, приведенная выше, является «строгой» (и, следовательно, безопасной) по следующим причинам:

- Она позволяет разработчику сайта при помощи одноразовых номеров (`'nonce-{RANDOM}'`) или хешей (`'sha256-{HASHED_INLINE_SCRIPT}'`) помечать «доверенные» теги `<script>`, выполнение которых должно быть разрешено в браузере пользователя.

- Она устанавливает параметр [`'strict-dynamic'`](https://www.w3.org/TR/CSP3/#strict-dynamic-usage), автоматически разрешающий выполнение скриптов, создаваемых уже доверенными скриптами, что позволяет упростить внедрение CSP-политики на основе одноразовых номеров или хешей. Это также делает возможным использование большинства сторонних библиотек и виджетов на основе JavaScript.

- Она не основана на списках разрешенных URL-адресов и поэтому не подвержена [стандартным методам обхода CSP](https://speakerdeck.com/lweichselbaum/csp-is-dead-long-live-strict-csp-deepsec-2016?slide=15).

- Она блокирует ненадежные встроенные скрипты, включая встроенные обработчики событий и URI с `javascript:`.

- Она ограничивает использование `object-src` и тем самым блокирует использование небезопасных плагинов, таких как Flash.

- Она ограничивает использование `base-uri`, тем самым запрещая внедрение тегов `<base>`, чтобы не позволить злоумышленникам изменять расположение скриптов, загружаемых при помощи относительных URL-адресов.

{% Aside %} Еще одно преимущество строгой CSP-политики заключается в том, что ее определение всегда имеет одинаковую структуру и его не нужно адаптировать для конкретного приложения. {% endAside %}

## Переход на строгую CSP-политику

Чтобы перейти на использование строгой CSP-политики, вам необходимо:

1. Определиться, какой вид CSP-политики ваше приложение будет использовать: на основе одноразовых номеров или на основе хешей.
2. Скопировать код CSP-политики из раздела [В чем заключается суть строгой политики безопасности контента](#what-is-a-strict-content-security-policy) и установить его в качестве заголовка ответа на всех страницах вашего приложения.
3. Провести рефакторинг HTML-шаблонов и кода, выполняемого на стороне клиента, чтобы исключить несовместимые с CSP паттерны.
4. Добавить резервные политики для поддержки Safari и устаревших браузеров.
5. Развернуть CSP-политику.

В ходе этого процесса вы можете использовать проверку **Best Practices** в [Lighthouse](https://developers.google.com/web/tools/lighthouse) (v7.3.0 или выше с флагом `--preset=experimental`) для подтверждения того, что на вашем сайте действует CSP-политика и она является достаточно строгой для предотвращения XSS-атак.

{% Img src="image/9B7J9oWjgsWbuE84mmxDaY37Wpw2/42a4iEEKsD4T3yU47vNQ.png", alt="Предупреждение отчета Lighthouse о том, что CSP-политика, применяемая в режиме исполнения, не обнаружена.", width="730", height="78" %}

### Шаг 1. Определитесь, какой вид CSP использовать: на основе одноразовых номеров или на основе хешей

Существует два вида строгих CSP-политик: на основе одноразовых номеров и на основе хешей. Вот как они работают:

- **CSP-политика на основе одноразовых номеров**: *во время выполнения* генерируется случайное число, которое указывается в коде CSP-политики и связывается с каждым тегом script на странице. Злоумышленник не сможет подключить к странице вредоносный скрипт и выполнить его, поскольку для этого ему пришлось бы угадать случайное число, соответствующее этому скрипту. Этот подход работает только в том случае, если число невозможно угадать и оно заново генерируется для каждого ответа на этапе выполнения.
- **CSP-политика на основе хешей**: в коде CSP-политики указывается хеш каждого встроенного тега script. Обратите внимание, что каждый скрипт имеет собственный хеш. Злоумышленник не сможет подключить к странице вредоносный скрипт и выполнить его, так как хеш скрипта должен присутствовать в коде CSP-политики.

Ниже приведены критерии для выбора типа строгой CSP-политики:

<div>
  <table>
      <caption>Критерии для выбора типа строгой CSP-политики</caption>
      <thead>
      <tr>
        <th>CSP-политика на основе одноразовых номеров</th>
        <td>Подходит для HTML-страниц, рендеринг которых происходит на сервере, благодаря чему случайный токен (одноразовый номер) можно генерировать заново для каждого ответа.</td>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>CSP-политика на основе хешей</th>
        <td>Подходит для HTML-страниц, загружаемых статически или с использованием кеша. Пример — одностраничные веб-приложения, построенные на таких фреймворках, как Angular, React и т. д., использующие статическую загрузку без рендеринга на стороне сервера.</td>
      </tr>
    </tbody>
  </table>
</div>

### Шаг 2. Установите строгую CSP-политику и подготовьте теги script

При установке CSP-политики необходимо определиться с парой моментов:

- Какой режим использовать: режим отправки отчетов (`Content-Security-Policy-Report-Only`) или режим исполнения (`Content-Security-Policy`). В режиме отправки отчетов CSP-политика не будет использоваться для блокирования ресурсов (т. е. все скрипты продолжат работать), однако вы сможете видеть ошибки и получать отчеты о ресурсах, попадающих под блокировку. Если вы настраиваете CSP-политику локально, между этими режимами нет существенной разницы: в обоих случаях вы будете видеть сообщения об ошибках в консоли браузера. В режиме исполнения выявлять заблокированные ресурсы и корректировать CSP-политику будет даже проще, так как вы сразу увидите, что страница функционирует неправильно. Наиболее полезным режим отправки отчетов становится на более поздних этапах процесса (см. [шаг 5](#step-5:-deploy-your-csp)).
- Где разместить политику: в заголовке или в HTML-теге `<meta>`. При локальной разработке использование тега `<meta>` может быть более уместным, так как позволяет с легкостью подстраивать CSP-политику и сразу же видеть результат. Однако имейте в виду следующее:
    - В дальнейшем, при развертывании CSP-политики на реальном сайте, рекомендуется использовать для ее установки HTTP-заголовок.
    - Если вы хотите использовать CSP-политику в режиме отправки отчетов, ее необходимо устанавливать при помощи заголовка: при установке CSP-политики при помощи тега meta режим отправки отчетов не поддерживается.

<span id="nonce-based-csp"></span>

{% Details %}

{% DetailsSummary %} Вариант А. CSP-политика на основе одноразовых номеров {% endDetailsSummary %}

Задайте для своего приложения следующий заголовок HTTP-ответа `Content-Security-Policy`:

```text
Content-Security-Policy:
  script-src 'nonce-{RANDOM}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';
```

{% Aside 'caution' %}

Вместо `{RANDOM}` должен указываться *случайный* одноразовый номер, генерируемый заново **при каждом ответе сервера**. {% endAside %}

#### Генерация одноразовых номеров для CSP-политики

Одноразовый номер — это случайное число, меняющееся при каждой загрузке страницы. CSP-политика на основе одноразовых номеров предотвращает XSS-атаки только в том случае, если значение номера **не может быть угадано** злоумышленником. Одноразовые номера, используемые в рамках CSP-политики, должны быть:

- **Криптографически стойкими** случайными числами (в идеале длиной от 128 бит).
- Генерируемыми **индивидуально для каждого ответа**.
- Закодированными при помощи Base64.

Вот примеры того, как указывать одноразовые номера для CSP-политики при использовании некоторых серверных фреймворков:

- [Django (Python).](https://django-csp.readthedocs.io/en/latest/nonce.html)
- Express (JavaScript):

```javascript
const app = express();
app.get('/', function(request, response) {
    // Генерация случайного одноразового числа заново для каждого ответа.
    const nonce = crypto.randomBytes(16).toString("base64");
    // Установка строгой CSP-политики на основе одноразовых чисел при помощи заголовка ответа
    const csp = `script-src 'nonce-${nonce}' 'strict-dynamic' https:; object-src 'none'; base-uri 'none';`;
    response.set("Content-Security-Policy", csp);
    // Это значение необходимо присвоить атрибуту `nonce` для каждого тега <script> в вашем приложении.
    response.render(template, { nonce: nonce });
  });
}
```

#### Добавление атрибута `nonce` к элементам `<script>`

При использовании CSP-политики на основе одноразовых номеров у каждого элемента `<script>` должен быть атрибут `nonce`, значение которого должно совпадать со случайным одноразовым номером, указанным в заголовке CSP-политики (все скрипты могут использовать один и тот же одноразовый номер). Для начала добавьте этот атрибут ко всем тегам script:

{% Compare 'worse', 'Запрещено CSP-политикой'%}

```html
<script src="/path/to/script.js"></script>
<script>foo()</script>
```

{% CompareCaption %} Эти скрипты будут заблокированы CSP-политикой, поскольку не имеют атрибута `nonce`. {% endCompareCaption %}

{% endCompare %}

{% Compare 'better', 'Разрешено CSP-политикой' %}

```html
<script nonce="${NONCE}" src="/path/to/script.js"></script>
<script nonce="${NONCE}">foo()</script>
```

{% CompareCaption %} Выполнение этих скриптов будет разрешено CSP-политикой, если заменить `${NONCE}` на значение, совпадающее с одноразовым номером, который указан в CSP-политике в заголовке ответа. Учтите, что некоторые браузеры скрывают атрибут `nonce` при инспектировании исходного кода страницы. {% endCompareCaption %}

{% endCompare %}

{% Aside 'gotchas' %} Если CSP-политика содержит параметр `'strict-dynamic'`, то одноразовые номера необходимо будет указывать только для тегов `<script>`, содержащихся в исходном HTML-ответе. Параметр `'strict-dynamic'` позволяет исполнять скрипты, динамически добавляемые на страницу, при условии, что их загрузка происходит из безопасного, уже одобренного скрипта (подробности см. в [спецификации](https://www.w3.org/TR/CSP3/#strict-dynamic-usage)). {% endAside %}

{% endDetails %}

<span id="hash-based-csp"></span>

{% Details %}

{% DetailsSummary %} Вариант Б. Заголовок ответа для CSP-политики на основе хешей {% endDetailsSummary %}

Задайте для своего приложения следующий заголовок HTTP-ответа `Content-Security-Policy`:

```text
Content-Security-Policy:
  script-src 'sha256-{HASHED_INLINE_SCRIPT}' 'strict-dynamic';
  object-src 'none';
  base-uri 'none';
```

При необходимости указать несколько встроенных скриптов используйте следующий синтаксис: `'sha256-{HASHED_INLINE_SCRIPT_1}' 'sha256-{HASHED_INLINE_SCRIPT_2}'` .

{% Aside 'caution' %} Вместо `{HASHED_INLINE_SCRIPT}` необходимо указать хеш (SHA-256, закодированный в base64) встроенного скрипта, используемого для загрузки остальных скриптов (см. следующий раздел). Для того чтобы вычислить SHA-хеш статического встроенного блока `<script>`, можно воспользоваться [этим инструментом](https://strict-csp-codelab.glitch.me/csp_sha256_util.html). В качестве альтернативного решения вы можете просмотреть в консоли разработчика Chrome предупреждения о нарушении CSP-политики и добавить содержащиеся в них хеши заблокированных скриптов в код политики, дописав к ним приставку «sha256-…».

Скрипты, внедряемые злоумышленниками, будут блокироваться, поскольку браузер будет разрешать выполнение только хешированного встроенного скрипта, а также скриптов, динамически подключаемых с его помощью. {% endAside %}

#### Динамическая загрузка внешних скриптов

Скрипты, подключаемые из внешних источников, необходимо загружать динамически при помощи встроенного скрипта, поскольку использование CSP-политики для проверки хешей внешних скриптов [поддерживается не во всех браузерах](https://wpt.fyi/results/content-security-policy/script-src/script-src-sri_hash.sub.html?label=master&label=experimental&aligned).

{% Img src="image/vgdbNJBYHma2o62ZqYmcnkq3j0o1/B2YsfJDYw8PRI6kJI7Bs.jpg", alt="", width="800", height="333" %}

{% Compare 'worse', 'Запрещено CSP-политикой' %}

```html
<script src="https://example.org/foo.js"></script>
<script src="https://example.org/bar.js"></script>
```

{% CompareCaption %} Эти скрипты будут заблокированы  CSP-политикой, поскольку хеширование поддерживается только для встроенных скриптов. {% endCompareCaption %}

{% endCompare %}

{% Compare 'better', 'Разрешено CSP-политикой' %}

```html
<script>
var scripts = [ 'https://example.org/foo.js', 'https://example.org/bar.js'];
scripts.forEach(function(scriptUrl) {
  var s = document.createElement('script');
  s.src = scriptUrl;
  s.async = false; // to preserve execution order
  document.head.appendChild(s);
});
</script>
```

{% CompareCaption %} Чтобы разрешить выполнение этого скрипта, необходимо вычислить его хеш и указать его в CSP-политике в заголовке ответа, заменив строку `{HASHED_INLINE_SCRIPT}`. При желании все встроенные скрипты можно объединить, чтобы сократить количество хешей. Чтобы увидеть, как это работает, вы можете ознакомиться с [примером](https://strict-csp-codelab.glitch.me/solution_hash_csp#) и изучить его [код](https://glitch.com/edit/#!/strict-csp-codelab?path=demo%2Fsolution_hash_csp.html%3A1%3A). {% endCompareCaption %}

{% endCompare %}

{% Aside 'gotchas' %} При вычислении хешей встроенных скриптов, указываемых в CSP-политике, необходимо учитывать символы пробелов между открывающим и закрывающим тегами `<script>`. Рассчитать хеш встроенного скрипта для CSP-политики можно при помощи [этого инструмента](https://strict-csp-codelab.glitch.me/csp_sha256_util.html). {% endAside %}

{% Details %}

{% DetailsSummary %} Примечание об использовании `async = false` при загрузке скриптов: в данном случае `async = false` не блокирует поток, но использовать его следует с осторожностью. {% endDetailsSummary %}

В приведенном выше фрагменте кода использование `s.async = false` гарантирует, что foo выполнится раньше, чем bar (даже если bar загрузится первым). <strong data-md-type="double_emphasis">В данном фрагменте использование `s.async = false` не блокирует парсер во время загрузки скриптов</strong>; это обусловлено тем, что скрипты добавляются динамически. Приостановка парсера будет происходить только во время выполнения скриптов — точно так же, как если бы параметр `async` был включен. Однако при рассмотрении этого фрагмента кода учитывайте следующее:

- Выполнение скриптов может начаться до того, как документ полностью загрузится. Если вы хотите, чтобы к моменту начала выполнения скриптов документ был уже загружен, необходимо дождаться [события `DOMContentLoaded`](https://developer.mozilla.org/docs/Web/API/Window/DOMContentLoaded_event) и уже после этого подключить скрипты. Если это приведет к падению производительности (из-за слишком позднего начала скачивания скриптов), вы можете указать [теги предварительной загрузки](https://developer.mozilla.org/docs/Web/HTML/Preloading_content) ближе к началу страницы.
- Не используйте параметр `defer = true`, поскольку он не даст нужного эффекта. Вместо этого вручную запускайте выполнение скрипта тогда, когда это требуется. {% endDetails %}

{% endDetails %}

### Шаг 3. Проведите рефакторинг HTML-шаблонов и кода, выполняемого на стороне клиента, чтобы исключить несовместимые с CSP паттерны

Встроенные обработчики событий (такие, как `onclick="…"` или `onerror="…"`) и URI, содержащие JavaScript (`<a href="javascript:…">`), могут использоваться для запуска скриптов. Это означает, что злоумышленник, обнаруживший XSS-уязвимость, сможет внедрить на страницу такой HTML-код и с его помощью выполнить вредоносный код JavaScript. CSP-политики на основе хешей и одноразовых номеров запрещают использование такой разметки. Если на вашем сайте используются описанные выше паттерны, необходимо провести рефакторинг и заменить их на более безопасные альтернативы.

Если в ходе предыдущего шага вы включили CSP, вы сможете видеть в консоли сообщения о нарушении CSP-политики каждый раз, когда происходит блокировка запрещенного паттерна.

{% Img src="image/vgdbNJBYHma2o62ZqYmcnkq3j0o1/mRWfNxAhQXzInOLCgtv8.jpg", alt="Сообщения о нарушении CSP-политики в консоли разработчика Chrome.", width="800", height="235" %}

В большинстве случаев исправление не потребует больших усилий:

#### Перепишите встроенные обработчики JavaScript таким образом, чтобы их добавление происходило из блока JavaScript

{% Compare 'worse', 'Запрещено CSP-политикой' %}

```html
<span onclick="doThings();">A thing.</span>
```

{% CompareCaption %} Встроенные обработчики событий будут заблокированы CSP-политикой. {% endCompareCaption%}

{% endCompare %}

{% Compare 'better', 'Разрешено CSP-политикой' %}

```html
<span id="things">A thing.</span>
<script nonce="${nonce}">
  document.getElementById('things')
          .addEventListener('click', doThings);
</script>
```

{% CompareCaption %} Регистрация обработчиков событий с помощью JavaScript разрешена CSP-политикой. {% endCompareCaption %}

{% endCompare %}

#### Для URI с `javascript:` можно использовать аналогичный паттерн

{% Compare 'worse', 'Запрещено CSP-политикой' %}

```html
<a href="javascript:linkClicked()">foo</a>
```

{% CompareCaption %} URI с javascript: будут заблокированы CSP-политикой. {% endCompareCaption %}

{% endCompare %}

{% Compare 'better', 'Разрешено CSP-политикой' %}

```html
<a id="foo">foo</a>
<script nonce="${nonce}">
  document.getElementById('foo')
          .addEventListener('click', linkClicked);
</script>
```

{% CompareCaption %} Регистрация обработчиков событий с помощью JavaScript разрешена CSP-политикой. {% endCompareCaption %}

{% endCompare %}

#### Использование `eval()` в JavaScript-коде

Если ваше приложение использует `eval()` для преобразования данных, сериализованных в виде JSON, в объекты JS, необходимо переписать такой код с использованием `JSON.parse()`: это не только безопаснее, но и [быстрее](https://v8.dev/blog/cost-of-javascript-2019#json).

Если полностью исключить использование `eval()` нет возможности, вы по-прежнему сможете использовать строгую CSP-политику на основе одноразовых номеров, однако вам придется добавить параметр `'unsafe-eval'`, который сделает политику немного менее безопасной.

С примерами такого рефакторинга вы можете ознакомиться в интерактивном уроке, посвященном строгим CSP-политикам: {% Glitch { id: 'strict-csp-codelab', path: 'demo/solution_nonce_csp.html', highlights: '14,20,28,39,40,41,42,43,44,45,54,55,56,57,58,59,60', previewSize: 35, allow: [] } %}

### Шаг 4. Добавьте резервные политики для поддержки Safari и устаревших браузеров

CSP-политики поддерживаются всеми основными браузерами, однако вам понадобится добавить две резервные политики:

- При использовании `'strict-dynamic'` необходимо указать `https:` в качестве резервной политики для поддержки Safari — единственного крупного браузера, не поддерживающего `'strict-dynamic'`. При использовании такой конфигурации:

    - Все браузеры, поддерживающие `'strict-dynamic'`, будут игнорировать `https:`, так что безопасность политики не снизится.
    - В Safari загрузка скриптов из внешних источников будет разрешена только для источников с протоколом HTTPS. Такая политика менее безопасна, чем строгая CSP-политика (поэтому она и является резервной), однако она по-прежнему защищает от многих распространенных причин XSS-атак, таких как внедрение URI с `javascript:`, поскольку при наличии хеша или одноразового номера параметр `'unsafe-inline'` игнорируется.

- Чтобы обеспечить совместимость с очень старыми версиями браузеров (вышедшими более 4 лет назад), вы можете добавить в качестве резервной политики `'unsafe-inline'`. Во всех современных браузерах при наличии CSP-политики на основе одноразового номера или хеша параметр `'unsafe-inline'` будет игнорироваться.

```text
Content-Security-Policy:
  script-src 'nonce-{random}' 'strict-dynamic' https: 'unsafe-inline';
  object-src 'none';
  base-uri 'none';
```

{% Aside %} Параметры `https:` и `unsafe-inline` не делают политику менее безопасной, поскольку будут игнорироваться браузерами, поддерживающими `strict-dynamic`. {% endAside %}

### Шаг 5. Разверните CSP-политику

Протестировав сайт в локальной среде разработки и убедившись, что CSP-политика не блокирует нужные скрипты, вы можете приступить к развертыванию CSP-политики сначала в промежуточной, а затем и в производственной среде:

1. (необязательно) Разверните CSP-политику в режиме отправки отчетов, используя заголовок `Content-Security-Policy-Report-Only` ([статья о Reporting API](/reporting-api/)). Режим отправки отчетов удобен для безопасного тестирования в производственной среде изменений, которые могут потенциально нарушить работу сайта, таких как новая CSP-политика. В режиме отправки отчетов CSP-политика не влияет на поведение приложения: сайт продолжает работать так же, как и до этого. Однако браузер будет отображать в консоли сообщения и отчеты о нарушении CSP-политики, когда встречает несовместимый с ней код (и таким образом позволит диагностировать проблемы, которые возникли бы у конечных пользователей).
2. Удостоверившись, что CSP-политика не вызовет нарушений в работе сайта у конечных пользователей, вы можете развернуть ее, используя заголовок ответа `Content-Security-Policy`. **Только после выполнения этого шага CSP-политика начнет защищать ваше приложение от XSS-атак**. Установка CSP-политики при помощи серверного HTTP-заголовка является более безопасной, чем использование тега `<meta>`; если у вас есть возможность, используйте заголовок.

{% Aside 'gotchas' %} Убедитесь, что используемая вами CSP-политика является «строгой», проверив ее при помощи [CSP Evaluator](https://csp-evaluator.withgoogle.com) или Lighthouse. Это очень важный шаг, поскольку даже небольшие изменения в политике могут значительно снизить ее безопасность. {% endAside %}

{% Aside 'caution' %} Активировав CSP-политику в производственной среде, вы можете увидеть в отчетах о нарушении CSP-политики некоторое количество шума, генерируемого браузерными расширениями и вредоносным ПО. {% endAside %}

## Ограничения

Как правило, строгая CSP-политика обеспечивает мощный дополнительный уровень защиты от XSS-атак. В большинстве случаев использование CSP-политики значительно снижает число уязвимых мест (полностью исключая использование таких опасных возможностей, как URI с `javascript:`). Однако в зависимости от типа применяемой CSP-политики (на основе одноразовых номеров, хешей, а также с использованием `'strict-dynamic'` или без него) существует ряд случаев, на которые ее действие не распространяется:

- Внедрение кода непосредственно в тело или в параметр `src` элемента `<script>`, защищенного одноразовым номером.
- Внедрение кода в места расположения динамически создаваемых скриптов (`document.createElement('script')`), включая библиотечные функции, создающие DOM-узлы `script` на основе передаваемых им аргументов. К таким функциям относятся некоторые распространенные API, такие как `.html()` в jQuery, а также `.get()` и `.post()` в jQuery &lt; 3.0.
- Внедрение шаблонов в старые приложения на AngularJS. Злоумышленник, внедривший шаблон AngularJS, может использовать его для [выполнения произвольного JavaScript-кода](https://sites.google.com/site/bughunteruniversity/nonvuln/angularjs-expression-sandbox-bypass).
- Внедрение кода в `eval()`, `setTimeout()` и ряд других редко используемых API,  если политика содержит параметр `'unsafe-eval'`.

Разработчикам и инженерам по безопасности следует обращать на такие моменты особое внимание в ходе проверок кода и аудитов безопасности. Подробнее об описанных выше случаях можно узнать в [этой презентации, посвященной CSP](https://static.sched.com/hosted_files/locomocosec2019/db/CSP%20-%20A%20Successful%20Mess%20Between%20Hardening%20and%20Mitigation%20%281%29.pdf#page=27).

{% Aside %} Trusted Types (доверенные типы) — отличное дополнение к строгим CSP-политикам, позволяющее преодолеть некоторые из ограничений, описанных выше. Подробности см. в [статье об использовании Trusted Types на web.dev](/trusted-types). {% endAside %}

## Материалы для дальнейшего чтения

- [CSP Is Dead, Long Live CSP! On the Insecurity of Whitelists and the Future of Content Security Policy](https://research.google/pubs/pub45542/)
- Инструмент [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [Content Security Policy — A successful mess between hardening and mitigation (конференция LocoMoco)](https://static.sched.com/hosted_files/locomocosec2019/db/CSP%20-%20A%20Successful%20Mess%20Between%20Hardening%20and%20Mitigation%20%281%29.pdf)
- [Securing Web Apps with Modern Platform Features (лекция с Google I/O)](https://webappsec.dev/assets/pub/Google_IO-Securing_Web_Apps_with_Modern_Platform_Features.pdf)
