# Lab 6 - <Д.Гантогтох>, <B242270139>

## Хэсэг A: Category-Partition

**Давуу эрэмбэ:** from → frozen → to → same → amount>0 → үлдэгдэл → лимит. Нэг хүсэлтэд хэд хэдэн алдаа байвал эхний шалгалтын алдаа буцна, учир нь функц шалгалтыг дарааллаар хийж эхний алдаан дээр зогсдог гэж үзсэн.

### A1. Сонголт ба төлөөлөх утга
| Параметр | Утга | Тайлбар |
|---|---|---|
| from | Valid / Frozen / Missing | байгаа царцаагүй / царцсан / байхгүй ID |
| to | Valid / Missing / SameAsFrom | байгаа / байхгүй / from-той ижил |
| amount | Normal / Zero / OverBalance / OverLimit | хэвийн / ≤0 / үлдэгдлээс их / өдрийн лимитээс их |
| currency | Same / Different | дансны валют / өөр (ханш хөрвүүлнэ) |
| Далд орчин | өдрийн лимит: хоосон / бараг дүүрсэн | амаар өгөөгүй ч OverLimit-д нөлөөлнө |

### A2. Хязгаарлалт
| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from=Frozen | [ERROR] | ERROR_FROZEN буцаана |
| from=Missing | [ERROR] | ERROR_NO_FROM буцаана |
| to=Missing | [ERROR] | ERROR_NO_TO буцаана |
| to=SameAsFrom | [ERROR] | ERROR_SAME_ACCOUNT буцаана |
| amount=Zero | [ERROR][SINGLE] | ≤0 нь бусад параметрээс үл хамаарна, нэг удаа хангалттай |
| amount=OverBalance | [ERROR] | ERROR_INSUFFICIENT |
| amount=OverLimit | [ERROR] | ERROR_LIMIT |
| currency=Different | [IF amount OverBalance/OverLimit] | хөрвүүлсний дараах дүнгээр шалгадаг тул хоёр утгатай хосолж тестлэнэ |

ERROR утга бүрийг бусад параметрийн хүчинтэй утгатай хосолж error masking-аас сэргийлнэ.

**N → M:** бүрэн олонлог 3 × 3 × 4 × 2 = **72**. Хязгаарлалтын дараа: happy 2 (currency 2 утга) + ERROR 6 утга × currency 2 = 12 + Zero [SINGLE] 1 = **15**. Тэгэхээр **72 → 15**.

### A3. Спецификацийн хүснэгт
| # | from | to | amount | currency | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | 201 OK |
| 2 | Valid | Valid | Normal | Different | 201 OK |
| 3 | Missing | Valid | Normal | Same | 200 ERROR_NO_FROM |
| 4 | Frozen | Valid | Normal | Same | 200 ERROR_FROZEN |
| 5 | Valid | Missing | Normal | Same | 200 ERROR_NO_TO |
| 6 | Valid | SameAsFrom | Normal | Same | 200 ERROR_SAME_ACCOUNT |
| 7 | Valid | Valid | Zero | Same | 200 ERROR_BAD_AMOUNT (SINGLE) |
| 8 | Valid | Valid | OverBalance | Same | 200 ERROR_INSUFFICIENT |
| 9 | Valid | Valid | OverLimit | Same | 200 ERROR_LIMIT |
| 10 | Valid | Valid | OverBalance | Different | 200 ERROR_INSUFFICIENT (хөрвүүлсний дараа) |
| 11 | Valid | Valid | OverLimit | Different | 200 ERROR_LIMIT (хөрвүүлсний дараа) |
| 12 | Missing | Missing | Zero | Same | 200 ERROR_NO_FROM (давхар, from эхэлж шалгагдана) |
| 13 | Frozen | SameAsFrom | OverBalance | Different | 200 ERROR_FROZEN (давхар) |
| 14 | Valid | Valid | OverBalance, лимит бараг дүүрсэн | Same | 200 ERROR_INSUFFICIENT (үлдэгдэл лимитээс өмнө) |

## Хэсэг B: PICT

Файлууд: `browser.model`, `transfer-nc.model`, `transfer.model`, `transfer-neg.model`. Гаралт нь `results/` дотор, `/r` ашиглаагүй.

### B1. Браузерын жишээ
| Хувилбар | Мөр | results файл |
|---|---|---|
| 2-way | 10 | browser-2way.txt |
| 3-way | 20 | browser-3way.txt |

**Greedy:** 2-way дээр PICT 10 мөр өгсөн, гараар хийсэн 9 мөрөөс нэгээр их. PICT нь мөр бүрт хамгийн олон шинэ хос бүрхэх мөрийг сонгож нэмдэг greedy алгоритм бөгөөд өмнө сонгосон мөрөө хойно эргэж засдаггүй. Тиймээс бүх хосыг бүрхсэн хамгийн цөөн олонлогийг олохыг баталгаажуулдаггүй, зөвхөн хурдан, хангалттай жижиг олонлог өгдөг. 3-way нь бүрэн олонлог 144-ийн оронд 20 мөр (ойролцоогоор 14%) болсон.

### B2. Шилжүүлгийн модель
| Хувилбар | Тестийн мөр | Happy path (Valid Valid Normal) | Алдааны утгын тоо |
|---|---|---|---|
| transfer-nc (хязгаарлалтгүй) | 13 | 1 | 0 алдаатай 1 мөр, 1 алдаатай 1 мөр, 2 алдаатай 8 мөр, 3 алдаатай 3 мөр |
| transfer (IF хязгаарлалттай) | 14 | 0 | Мөр бүрд 1 алдааны нөхцөл |
| transfer-neg (~ сөрөг утга) | 16 | 2 | 14 мөрд 1 сөрөг утга, 2 мөрд сөрөг утга байхгүй |

- **Хязгаарлалттай нь 1 мөрөөр их (13 → 14)**: хязгаарлалт нь алдааны утгуудын хоорондын хосыг хүчингүй болгосон тул үлдсэн хосыг greedy өөрөөр бүрхсэн. Энэ нь алдаа биш.
- **transfer.txt-д happy path мөр байхгүй**: pairwise нь Valid-Valid-Normal гурвалыг нэг мөрөнд цуглуулах албагүй, зөвхөн хос бүрийг бүрхдэг. Тиймээс happy path-ийг тусад нь (A3 хүснэгтийн 1-2-р мөр) тестлэх хэрэгтэй.
- **transfer-nc**: алдааны утгуудыг хооронд нь хосолсон тул нэг мөрөнд хэд хэдэн алдаа орж error masking үүснэ, ямар алдаа буцахыг oracle-оор тодорхойлох хэцүү.
- **~ хэрэглэхэд хязгаарлалт хэрэгтэй юу**: error masking-ийг сэргийлэх `IF ... THEN` хязгаарлалт хэрэггүй болсон, учир нь PICT нэг мөрөнд хамгийн ихдээ нэг сөрөг утга оруулдаг. Харин currency=Different ба Over* хоёрын харилцан хамаарал (хөрвүүлсний дараа шалгах) PICT-д илэрхийлэгдэхгүй, гэхдээ `~OverBalance/~OverLimit` хоёулаа Different-тэй хосолсон мөр гарсан (neg-ийн 3, 9-р мөр) тул хамралт хангагдсан.

### Бодит тест кейсүүд
| # | Эх мөр | Setup | Оролт | Oracle |
|---|---|---|---|---|
| 1 (happy) | neg: Valid Valid Normal Same | A: үлдэгдэл 1 000 000 MNT, царцаагүй, өдрийн лимит 500 000 MNT, өнөөдөр 0 зарцуулсан; B: байгаа, MNT | `{"from":A,"to":B,"amount":100000,"currency":"MNT"}` | 201 `{"result":"OK"}` |
| 2 (нэг алдаа) | neg: ~Frozen Valid Normal Same | A: царцаасан; B: байгаа, MNT | `{"from":A,"to":B,"amount":100000,"currency":"MNT"}` | 200 `{"result":"ERROR_FROZEN"}` |
| 3 (currency + лимит) | transfer: Valid Valid OverLimit Different | A: үлдэгдэл 10 000 000 MNT, царцаагүй, өдрийн лимит 1 000 000 MNT, 0 зарцуулсан; B байгаа; ханш 1 USD = 3500 MNT гэж үзсэн | `{"from":A,"to":B,"amount":1000,"currency":"USD"}` | 200 `{"result":"ERROR_LIMIT"}`: 1000 USD = 3 500 000 MNT > лимит 1 000 000, үлдэгдэл 10 000 000 хүрэлцээтэй тул ERROR_INSUFFICIENT биш |

Ганц алдаатай мөр сонгосон тул давуу эрэмбэнд найдах шаардлагагүй. Эрэмбэ нь A хэсэгт тогтоогдсон (from → frozen → to → same → amount>0 → үлдэгдэл → лимит).

## B3. Дүгнэлт
Category-partition гараар хийхэд сонголт бүрийн үндэслэл, хүлээгдэх алдаа, давхар нөхцөлийн давуу эрэмбийг өөрөө бодож гаргадаг тул алдааны семантикийг сайн барьдаг. Үүний оронд гараар хийсэн A3 хүснэгт хэдхэн мөртэй тул хосын хамралтыг баталгаажуулдаггүй. PICT хос бүрийг бүрхсэнийг алгоритмаараа баталгаажуулж, 72 хослолын оронд 13-16 мөр өгсөн нь тест хэмнэнэ. Гэхдээ PICT алдааны утгыг энгийн утга шиг хосолбол нэг мөрөнд хэд хэдэн алдаа орж error masking үүсэх тул `~` сөрөг утга эсвэл IF хязгаарлалт заавал хэрэгтэй. Мөн хязгаарлалттай хувилбарт happy path мөр гарахгүй байж болохыг transfer.txt харуулсан. PICT нь greedy тул хамгийн цөөн мөрийг өгөхгүй, браузерын жишээнд 9-ийн оронд 10 мөр гарсан. Мөн PICT oracle өгдөггүй, хүлээгдэх үр дүнг хүн өөрөө бодож бичих ёстой. Тиймээс хоёр аргыг хослуулж, A-аар утга ба хязгаарлалтаа гаргаад B-ээр хамралтыг шалгах нь хамгийн сайн. Хамгийн их бодол шаардсан алхам нь A2 буюу ямар утгыг ERROR/SINGLE/IF гэж үндэслэж ангилах, мөн currency-гийн хөрвүүлэлт ба amount-ын харилцан хамаарлыг тооцох хэсэг байлаа.
