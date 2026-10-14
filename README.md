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
