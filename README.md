# Selective VPN Routing for Happ

Инструкция по настройке **split-routing** в Happ, чтобы VPN использовался **только для выбранных сайтов** (например: YouTube, Discord, AI-сервисы), а остальной трафик шел напрямую.

Документация Happ:
[https://www.happ.su/main/ru/dev-docs](https://www.happ.su/main/ru/dev-docs)

---
## Предисловие

Была ли у вас ситуация, когда из-за блокировок на компьютере стоит **100500 способов обхода**, и один начинает наслаиваться на другой? В итоге соединение с нужными сайтами становится только медленнее.

У меня такая проблема появилась довольно быстро. Сидишь с друзьями в Discord, смотришь видео на YouTube — всё работает нормально. И тут вдруг вспоминаешь, что **забыл сдать какой-нибудь конспект по философии**, а дедлайн через час.

Включаешь VPN, чтобы зайти на сайт нейросети.

И в этот момент начинается:

- голос в Discord начинает лагать
    
- видео на YouTube грузятся медленнее
    
- VPN + zapret начинают конфликтовать
    
- а сам сайт нейросети всё равно открывается медленно
    

Потому что VPN начинает **гнать через себя вообще весь трафик**, включая то, что прекрасно работало и без него.

В какой-то момент стало понятно, что проблема не в скорости VPN, а в том, **что используется неправильная маршрутизация трафика**. По сути, VPN нужен только для нескольких сайтов, а не для всего интернета.

Тогда появилась идея сделать **split-routing** через Happ — чтобы VPN работал **только для нужных доменов**, а остальной трафик шёл напрямую.

Я полез в официальную документацию Happ. Она вроде и на русском, но читается… как будто на арабском. В итоге, после нескольких часов разбора JSON-правил и routing-схем, удалось собрать рабочую конфигурацию.

Этот репозиторий — мой **первый небольшой гайд**, который показывает:

- как настроить routing-правила для Happ
    
- как сделать VPN только для нужных сайтов
    
- как не убивать скорость YouTube, Discord и других сервисов
    

Если у вас такая же ситуация с кучей обходов и медленным интернетом — этот небольшой гайд может сэкономить немного времени и нервов.
---
# Что это делает

После настройки:

| Трафик          | Маршрут  |
| --------------- | -------- |
| YouTube         | VPN      |
| Discord         | VPN      |
| Twitter / X     | VPN      |
| AI сервисы      | VPN      |
| Остальные сайты | напрямую |

Это уменьшает задержки и снижает нагрузку на VPN.

---

# Требования

* установленный клиент **Happ**
* поддержка **routing profiles**
* возможность открыть ссылки формата:

```
happ://routing/...
```

---

# Шаг 1 — создать routing-профиль

Создайте файл `routing.json`. Или используйте его за основу.

```json
{
  "Name": "Selective VPN",
  "GlobalProxy": false,
  "RouteOrder": "proxy-direct-block",

  "ProxySites": [
    "youtube.com",
    "youtu.be",
    "googlevideo.com",
    "ytimg.com",

    "discord.com",
    "discord.gg",
    "discordapp.com",
    "discordapp.net",

    "twitter.com",
    "x.com",
    "twimg.com",

    "openai.com",
    "chat.openai.com",
    "anthropic.com",
    "claude.ai",
    "perplexity.ai",

    "huggingface.co",
    "replicate.com"
  ],

  "DirectSites": [],
  "BlockSites": [],

  "RemoteDNSType": "DoH",
  "RemoteDNSDomain": "https://cloudflare-dns.com/dns-query",
  "RemoteDNSIP": "1.1.1.1",

  "DomesticDNSType": "DoH",
  "DomesticDNSDomain": "https://dns.google/dns-query",
  "DomesticDNSIP": "8.8.8.8"
}
```
>Также вы можете добавить интересующие вас сайты вида `example.com` в `ProxySites`
---

# Шаг 2 — как работает конфигурация

Главный параметр:

```
GlobalProxy = false
```

Это означает:

* весь трафик идет **напрямую**
* **только сайты из ProxySites используют VPN**

Порядок обработки правил:

```
proxy → direct → block
```

---

# Шаг 3 — кодирование профиля

Happ принимает routing-профили через **Base64-ссылку**.

### Способ 1. Сайт
Через сайт [https://routing.happ.su/ru](https://routing.happ.su/ru)
1. Вставляете скопированный JSON из шага 1 в графу JSON‑редактор
2. Нажимаете "Применить"
3. В блоке "Сгенерированная ссылка" можете скопировать ссылку и открыть в новой вкладке(или же воспользоваться QR-кодом), браузер сам предложит вам открыть Happ и автоматически внесет все данные
4. Радоваться жизни

### Способ 2. Через кодировку(мудрёнее)
Linux / macOS

```bash
base64 routing.json
```

Windows (PowerShell)

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes((Get-Content routing.json -Raw)))
```


После кодирования создайте ссылку:

```
happ://routing/onadd/{base64}
```

где `{base64}` — закодированный JSON.

Пример:

```
happ://routing/onadd/eyJOYW1lIjoiU2VsZWN0aXZlIFZQTiIsIkdsb2JhbFByb3h5IjpmYWxzZX0=
```

---

# Шаг 5 — добавить профиль в Happ

1. скопируйте ссылку `happ://routing/onadd/...`
2. откройте её в браузере
3. Happ автоматически:

   * добавит профиль
   * активирует routing

---

# Расширенный список сайтов (рекомендуется)

Чтобы избежать проблем с CDN и загрузкой медиа:

```
youtube.com
youtu.be
googlevideo.com
ytimg.com

discord.com
discord.gg
discordapp.com
discordapp.net

twitter.com
x.com
twimg.com

openai.com
chat.openai.com
anthropic.com
claude.ai
perplexity.ai

huggingface.co
replicate.com

github.com
githubusercontent.com

reddit.com
redd.it
imgur.com

medium.com
```

---

# Полезные советы

### Добавление сайта в VPN

Добавьте домен в `ProxySites`.

```
"example.com"
```

---

### Прямое соединение для сайта

Добавьте в `DirectSites`.

```
"example.com"
```

---

### Блокировка сайта

Добавьте в `BlockSites`.

```
"example.com"
```

---

# Структура routing-профиля

```
routing.json
README.md
```

---
