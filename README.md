## Selective VPN Routing for Happ

Инструкция по настройке **split-routing** в Happ, чтобы VPN использовался **только для выбранных сайтов** (например: YouTube, Discord, AI-сервисы), а остальной трафик шел напрямую.

Документация Happ:
[https://www.happ.su/main/ru/dev-docs](https://www.happ.su/main/ru/dev-docs)

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

Создайте файл `routing.json`.

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
2. Нажимаете применить
3. В блоке "Сгенерированная ссылка" можете скопировать запрос и открыть в новой вкладке, браузер сам предложит вам открыть Happ и автоматически внесет все данные
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


---

# Шаг 4 — создать ссылку для импорта

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
