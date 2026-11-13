# stock-trade-skill

[![tests](https://github.com/devfn93/stock-trade-skill/actions/workflows/tests.yml/badge.svg)](https://github.com/devfn93/stock-trade-skill/actions/workflows/tests.yml)

[العربية](#بالعربي) · [English](#english)

Eleven Claude skills for **US stocks**: copy an investor's 13F portfolio, watch
the news and Congress's trade disclosures, read earnings, screen companies,
size every position to your budget, check every order against your limits,
track tax lots, ring price alerts and rebalance.
**Claude never places an order: you do, in your own broker.**

Free, MIT, no signup, no API keys. Plain Python 3.8+ (standard library only)
reading public data: SEC EDGAR, the House Clerk, news feeds.

---

## بالعربي

مكتبة مهارات لـ Claude للأسهم الأمريكية: تنسخ محفظة مستثمر من تقارير 13F، وتتابع الأخبار وصفقات الكونغرس، وتقرا أرباح الشركات، وتفلتر الأسهم، وتوزع الميزانية، وتفحص كل أمر على حدودك.

**Claude ما ينفذ أي أمر.** يطلعلك تذكرة جاهزة وانت تدخلها بالبروكر مالتك.

| المهارة | شنو تسوي |
| --- | --- |
| `/trader-copycat` | تبني محفظة مستثمر مشهور من تقرير 13F، وشنو اشترى وباع من الربع اللي قبله |
| `/trader-news` | الأخبار وإفصاحات 8-K للأسهم اللي عندك |
| `/trader-budget` | توزع الميزانية مع احتياطي كاش وحد أعلى لكل سهم |
| `/trader-pelosi` | صفقات الأسهم اللي يعلنها أعضاء الكونغرس |
| `/trader-analyst` | الإيرادات والهوامش وربح السهم من تقارير SEC |
| `/trader-screener` | فلترة الشركات حسب شروطك وترتيبها |
| `/trader-risk` | فحص كل أمر على حدودك: PASS أو BLOCK |
| `/trader-broker` | تذكرة الأمر اللي تدخلها انت، وسجل التنفيذ |
| `/trader-tax` | الأرباح والخسائر، وحصاد الخسائر الضريبية (نظام أمريكي) |
| `/trader-alerts` | تنبيهات الأسعار ومواعيد بورصة نيويورك |
| `/trader-rebalance` | إعادة المحفظة لنسبها المستهدفة |

### البداية السريعة

1. انسخ `trader-budget/config.example.json` إلى `trader/config.json` واكتب ميزانيتك وحدودك، واسمك وإيميلك بخانة `sec_user_agent` لأن SEC تطلب تعريف كل طلب.
2. اطلب من Claude: "استعمل trader-copycat وانسخ أكبر 5 أسهم ببيركشاير".
3. بعدها: trader-budget ثم trader-risk ثم trader-broker يطلعلك التذاكر.

---

## English

| command | job |
| --- | --- |
| `/trader-copycat` | Rebuilds an investor's portfolio from their 13F-HR, with what they bought, added, trimmed and sold |
| `/trader-news` | Headlines and 8-K filings for every stock you hold |
| `/trader-budget` | Sizes positions to your budget, with a cash buffer and a per-stock cap |
| `/trader-pelosi` | House Periodic Transaction Reports turned into clean rows, one alert per new filing |
| `/trader-analyst` | Revenue, margins, net income and diluted EPS from SEC XBRL, quarter by quarter |
| `/trader-screener` | Screens companies against your own rules and ranks what passes |
| `/trader-risk` | Checks every proposed order against your limits: PASS or BLOCK with the reason |
| `/trader-broker` | The ticket you type into your own broker, your decision and the fill log |
| `/trader-tax` | Cost-basis lots, gains and losses, tax-loss harvesting, wash sales (US) |
| `/trader-alerts` | Price, stop and percent-move alerts with the real NYSE calendar |
| `/trader-rebalance` | Back to target weights when the portfolio drifts past your band |

### Quick start

```bash
mkdir -p trader
cp trader-budget/config.example.json trader/config.json   # then edit it
```

Set `"sec_user_agent": "Your Name you@example.com"` in `trader/config.json` (or
the `SEC_USER_AGENT` environment variable): SEC asks every script to identify itself.

### Flows

```
trader-copycat -> trader-budget -> trader-risk -> trader-broker            copy an investor
trader-analyst -> trader-screener -> trader-budget -> trader-risk -> trader-broker   pick your own
trader-news / trader-pelosi / trader-alerts                                watch
trader-rebalance / trader-tax -> trader-risk -> trader-broker              maintain
```

Everything lands in a `trader/` folder in your project, and each skill reads
what the previous one wrote.

### Configuration

| key | default | meaning |
| --- | --- | --- |
| `budget` | `5000` | Money this is for. Only money you can afford to lose |
| `cash_buffer_pct` | `10` | Percent kept in cash |
| `max_position_pct` | `25` | Largest share of the budget in one stock |
| `max_order_usd` / `min_order_usd` | `2000` / `5` | Largest and smallest single order |
| `max_orders_per_day` | `10` | Orders per day |
| `fractional` | `true` | Fractional shares; `false` for whole shares |
| `paper` | `true` | Paper account |
| `blocked` | `[]` | Tickers you never want |
| `allow_options` / `allow_margin` / `allow_short` | `false` | Off unless you turn them on |
| `sec_user_agent` | — | Your name and email for SEC requests |

### The tools

```bash
python3 trader-copycat/edgar13f.py find buffett
python3 trader-copycat/edgar13f.py filings 1067983
python3 trader-news/news.py --tickers AAPL AXP GOOGL KO BAC --hours 24
python3 trader-pelosi/ptr.py index --member pelosi
python3 trader-analyst/facts.py AAPL
python3 trader-screener/screen.py trader/metrics.json --rules trader/rules.json
python3 trader-budget/size.py --budget 5000 --write
python3 trader-risk/check.py
python3 trader-broker/ticket.py show
python3 trader-tax/lots.py harvest --min-loss 100
python3 trader-alerts/alerts.py clock
python3 trader-rebalance/rebalance.py --band 5
```

### Install

**Claude (web, desktop, mobile):** zip a skill folder (the folder at the root of
the zip) and upload it in **Customize > Skills > + > Upload a skill**.

**Claude Code, as a plugin:**

```
/plugin marketplace add devfn93/stock-trade-skill
/plugin install stock-trade-skill@stock-trade-skill
```

Or copy the folders: `cp -r trader-* ~/.claude/skills/`

### Tests

```bash
python3 -m unittest discover -s tests -v
```

Offline, on fixtures cut from real filings and feeds.

## Fine print

**Not financial advice.** Software that reads public data and does arithmetic.
It does not know your situation and cannot see the future. You can lose money.

**No order is ever placed by Claude or by these tools.** `/trader-broker` shows
a ticket; you place it in your broker. Never share broker keys or passwords.

**13F data is old and partial**: filed up to 45 days after each quarter, long US
positions only. **Congress disclosures** are filed up to 45 days after the
trade, as amount ranges. **Free quotes are delayed.** Check against your broker.

**Taxes**: `/trader-tax` is arithmetic on US rules, not tax advice.

## Author

**Esraa Fadhil Naif** · [github.com/devfn93](https://github.com/devfn93)

MIT. See [LICENSE](LICENSE).
