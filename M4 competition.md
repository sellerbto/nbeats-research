

# M4 Competition — конспект на 1 минуту

> [!summary] Коротко
> M4 — крупнейшее соревнование серии Makridakis Competitions: **100 000 временных рядов**.  
> Главный вывод: **гибриды и комбинации моделей** снова оказались сильнее большинства “чистых” ML-подходов. Overall победил **Smyl** с гибридом **exponential smoothing + RNN**.

---

## Контекст

M4 продолжала серию **Makridakis Competitions**:

| Competition | Количество рядов |
|---|---:|
| M1 | 111 |
| M2 | 23 |
| M3 | 3 003 |
| M4 | **100 000** |

Главный урок прошлых соревнований:

> Простые statistical methods и combinations часто бьют “сложные” модели.

В **M3** выиграл **Theta method**, а в **M4** — гибрид **Smyl**:  
**exponential smoothing + RNN**.

---

## Что было в M4

В M4 участникам дали **100 000 continuous business time series**.

Категории были устроены двумя способами:

1. по **доменам**;
2. по **частоте**.

---

### Домены

| Категория | Что примерно означает | Кол-во рядов |
|---|---|---:|
| **Micro** | Микроуровень: отдельные продукты, компании, локальные бизнес-показатели, спрос, продажи и т.п. | 25 121 |
| **Industry** | Отраслевые ряды: производство, сектора экономики, индустриальные показатели. | 18 798 |
| **Macro** | Макроэкономика: GDP-like показатели, инфляция, занятость, государственные / страновые агрегаты. | 19 402 |
| **Finance** | Финансовые временные ряды: акции, индексы, облигации, кредиты, страхование и похожие данные. **Не HFT**. | 24 534 |
| **Demographic** | Демография: население, возрастные группы, миграция, birth / death-like статистика. | 8 708 |
| **Other** | Всё, что не попало в остальные домены. В частности, все **hourly** ряды в M4 были именно здесь. | 3 437 |

---

### Частоты

| Частота | Что это | Кол-во рядов |
|---|---|---:|
| **Yearly** | Годовые данные, long-term forecasting. | 23 000 |
| **Quarterly** | Квартальные данные, часто budget / macro / business planning. | 24 000 |
| **Monthly** | Месячные данные, самая большая часть M4. | 48 000 |
| **Weekly** | Недельные данные. | 359 |
| **Daily** | Дневные данные. | 4 227 |
| **Hourly** | Почасовые данные, но не market microstructure / HFT. | 414 |

---

> [!note] Главный перекос датасета
> M4 в основном про **business / economic forecasting**, поэтому больше всего было **monthly**, **finance**, **micro**, а также **quarterly / yearly** рядов.

Отдельно важно:

- **Finance** был отдельной категорией — **24 534 ряда**.
- “Добыча” и “погода” не были отдельными категориями.
- Если такие сюжеты и были, они скорее попадали в **Industry** или **Other**.
- Почасовые ряды были, но их было мало: всего **414**, и все они относились к **Other**.

---

## Почему не было минуток / HFT

Самая частая гранулярность в M4 — **hourly**, всего **414 рядов**.

M4 была про **generic business forecasting**, а не про **market microstructure**.

HFT требует других данных и другой постановки задачи:

- order book / tick data
- latency
- transaction costs
- PnL metrics

А M4 оценивала не торговую стратегию, а **forecasting accuracy** на публичных continuous time series с hidden test set.

> [!note] Интерпретация
> Это моя интерпретация из дизайна M4: public continuous time series, hidden test set, business domains.

---

## Метрики

Для **point forecasts** методы ранжировали по **OWA**.

### sMAPE

$$
sMAPE =
\frac{2}{h}
\sum_{t=n+1}^{n+h}
\frac{|Y_t-\hat{Y}_t|}{|Y_t|+|\hat{Y}_t|}
\cdot 100
$$

### MASE

$$
MASE =
\frac{
\frac{1}{h}
\sum_{t=n+1}^{n+h}
|Y_t-\hat{Y}_t|
}{
\frac{1}{n-m}
\sum_{t=m+1}^{n}
|Y_t-Y_{t-m}|
}
$$

### OWA

$$
OWA =
\frac{1}{2}
\left(
\frac{sMAPE}{sMAPE_{Naive2}}
+
\frac{MASE}{MASE_{Naive2}}
\right)
$$

---

## Uncertainty forecasting

Для uncertainty участники должны были дать **95% prediction interval**:

$$
PI_{95\%} = [q_{0.025}, q_{0.975}]
$$

Оценивали интервалы через **MSIS**.

MSIS штрафует за две вещи:

1. слишком широкий interval;
2. промах фактического значения вне interval.

---

## Сколько длилось соревнование

Данные открыли:

- **31 декабря 2017** / старт — **1 января 2018**
- дедлайн — **31 мая 2018**

Итого: примерно **5 месяцев**.

---

## Как проходил evaluation

Test set был скрыт до конца соревнования.

После дедлайна организаторы раскрыли future values и посчитали:

- sMAPE
- MASE
- OWA
- MSIS

Валидные submissions:

| Тип submission | Количество |
|---|---:|
| Point forecasts | 49 |
| Prediction intervals | 20 |

---

## Где был лучший performance

Overall победил **Smyl**:

$$
OWA = 0.821
$$

Главный паттерн:

> Лучше всего работали **hybrid models** и **combinations**.  
> Хуже — pure ML-подходы.

По разбивкам official results дают scores / ranks по:

- frequency
- domain
- total score

Сильный relative performance был у **hourly / Other**, но это маленький сегмент.

Поэтому главный вывод лучше формулировать так:

> **Smyl был самым универсальным, а combinations / hybrids доминировали почти везде.**

Источник: [M4-methods GitHub][1]

---

## Читеры и защита от leakage

В статье нет истории про читеров или “hack”.

Наоборот, организаторы:

- убрали starting dates;
- убрали идентифицирующую информацию;
- держали test set закрытым;
- поощряли reproducibility через код.

Ближайший риск, который обсуждали, — **overfitting к старому M3 benchmark**.

Именно поэтому M4 сделали намного больше.

---

## Главный takeaway

> [!important]
> M4 показала, что для large-scale forecasting универсальные гибриды и combinations часто надежнее, чем попытка применить один “сильный” ML-метод ко всем рядам.

---

[1]: https://github.com/Mcompetitions/M4-methods "GitHub - Mcompetitions/M4-methods: Data, Benchmarks, and methods submitted to the M4 forecasting competition · GitHub"

## Benchmark’и M4

В M4 использовали несколько baseline / benchmark методов для сравнения submitted methods.

---

### Point forecast benchmark’и

| Benchmark | Идея |
|---|---|
| **Naïve 1** | Random walk: будущие значения равны последнему наблюдению. |
| **Naïve S** | Seasonal naïve: прогноз равен последнему наблюдению из такого же сезона / периода. Например, для monthly — значение того же месяца прошлого года. |
| **Naïve 2** | Как **Naïve 1**, но перед этим ряд сезонно корректируется, если есть сезонность: **deseasonalize → random walk forecast → reseasonalize**. Именно относительно **Naïve 2** считали **OWA**. |
| **SES** | Simple Exponential Smoothing: сглаживает уровень ряда, без тренда. Хорош для рядов без явного тренда. |
| **Holt** | Exponential smoothing с линейным трендом: отдельно оценивает **level** и **trend**. |
| **Damped** | Holt с “затухающим” трендом: тренд есть, но его влияние уменьшается на горизонте. Часто стабильнее обычного Holt. |
| **Theta** | Метод из M3: раскладывает ряд на две Theta-линии, одну экстраполирует linear regression, другую — SES, потом усредняет. |
| **Comb** | Простое среднее трёх методов: **SES**, **Holt** и **Damped**. В M4 использовался как главный простой statistical benchmark для сравнения submitted methods. |
| **MLP** | Простой multilayer perceptron benchmark. Перед ML делали preprocessing: **detrending / deseasonalization**. |
| **RNN** | Простой recurrent neural network benchmark. Тоже с preprocessing. Был нужен как базовый ML benchmark. |
| **ETS** | Автоматический выбор лучшей exponential smoothing модели по information criteria. Это не benchmark-призёр, а standard for comparison. |
| **ARIMA** | Автоматический выбор ARIMA-модели по selection criteria. Тоже standard for comparison. |

---

### Формула Naïve 1

$$
\hat{Y}_{t+h} = Y_t
$$

---

### Prediction interval benchmark’и

Для **prediction intervals** использовали отдельные benchmark’и.

| PI benchmark | Идея |
|---|---|
| **Naïve 1** | Главный benchmark для интервалов: random walk + uncertainty interval. |
| **ETS** | Интервалы из автоматически выбранной ETS-модели. |
| **ARIMA** | Интервалы из автоматически выбранной ARIMA-модели. |

---

## Самое важное

> [!important]
> Для понимания M4 важнее всего помнить три benchmark’а:
>
> - **Naïve 2** — baseline для **OWA**.
> - **Comb** — сильный простой statistical benchmark.
> - **Naïve 1** — baseline для **prediction intervals**.

---

## Короткая логика benchmark’ов

| Метод           | Роль в M4                                            |
| --------------- | ---------------------------------------------------- |
| **Naïve 2**     | Главная точка отсчёта для point forecasts через OWA. |
| **Comb**        | Простая, но сильная statistical комбинация.          |
| **Naïve 1**     | Главная точка отсчёта для prediction intervals.      |
| **ETS / ARIMA** | Классические standards for comparison.               |
| **MLP / RNN**   | Базовые ML-бенчмарки, не главные победители.         |