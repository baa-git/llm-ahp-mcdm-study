# LLM–AHP deneyi: teknik rapor

Bu rapor `python -m ahp_bench.tech_report` komutuyla kayıt dosyalarından üretilmiştir. Rapordaki sayılar `results/raw/` altındaki JSONL dosyalarından, `results/report/tables/` tablolarından ve `results/report/appendix/` eklerinden program tarafından okunmuş ya da hesaplanmıştır; elle yazılmamıştır. Ondalık ayırıcı olarak nokta kullanılmıştır. Zamanlar UTC'dir.

## 1. Amaç ve kapsam

Deneyde 9 büyük dil modeline (LLM) OpenRouter üzerinden tedarikçi değerlendirme ölçütleri için AHP ikili karşılaştırma soruları soruldu. Amaç, modellerin verdiği yanıtlardan AHP ağırlıklarını hesaplamak, bunları aynı soruları yanıtlayan 7 uzmanın ağırlıklarıyla karşılaştırmak ve ağırlıkların tedarikçi sıralamasına etkisini göstermektir.

Ana deney (`run_tag = main`) 270 çağrıdan oluşur: 9 model × 3 koşul × 2 görev × 5 tekrar. Ek olarak yönlendirmesiz – doğrudan (direct) koşul iki farklı sıcaklıkta tekrarlandı ve tutarsız matrisler için bir hakem modelle revizyon aşaması yürütüldü. Aşamaların kayıt tarihleri aşağıdadır.

| Aşama | Dosya | run_tag | Kayıt | İlk kayıt | Son kayıt |
| --- | --- | --- | --- | --- | --- |
| Ön deneme (pilot, sonuçlara dahil değil) | calls.jsonl | pilot | 10 | 2026-11-23T16:55:23+00:00 | 2026-11-23T16:56:13+00:00 |
| Ana deney | calls.jsonl | main | 270 | 2026-11-23T17:05:25+00:00 | 2026-11-23T18:22:20+00:00 |
| Sıcaklık kolu temp_0.0 | calls.jsonl | temp_0.0 | 54 | 2026-11-23T18:25:58+00:00 | 2026-11-23T18:29:34+00:00 |
| Sıcaklık kolu temp_1.0 | calls.jsonl | temp_1.0 | 54 | 2026-11-23T18:29:36+00:00 | 2026-11-23T18:33:57+00:00 |
| Revizyon pilotu revision_judge (panel, dahil değil) | revisions.jsonl | revision_judge | 245 | 2026-11-23T18:23:36+00:00 | 2026-11-23T18:56:40+00:00 |
| Revizyon pilotu revision_v2 (panel, dahil değil) | revisions.jsonl | revision_v2 | 6 | 2026-11-23T19:32:39+00:00 | 2026-11-23T19:36:28+00:00 |
| Revizyon pilotu rev_final (panel, dahil değil) | revisions.jsonl | rev_final | 7 | 2026-11-23T19:37:50+00:00 | 2026-11-23T19:41:51+00:00 |
| Revizyon pilotu rev_test_none (panel, dahil değil) | revisions.jsonl | rev_test_none | 2 | 2026-11-23T19:42:35+00:00 | 2026-11-23T19:43:14+00:00 |
| Revizyon (revision_final), panel çağrıları | revisions.jsonl | revision_final | 250 | 2026-11-23T19:43:50+00:00 | 2026-11-23T20:23:11+00:00 |
| Revizyon (revision_final), hakem çağrıları | judge_calls.jsonl | revision_final | 500 | 2026-11-23T19:43:48+00:00 | 2026-11-23T20:23:14+00:00 |

## 2. Deney tasarımı

### 2.1 Modeller ve ayarlar

İstenen sıcaklık: 0.6. Akıl yürütme (reasoning) sağlayıcı izin verdiğinde kapatıldı (`effort=none`). Tablo, ana deney kayıtlarındaki istenen ve uygulanan değerleri gösterir.

| Model | Geliştirici | OpenRouter kimliği | Sıcaklık (istenen) | Sıcaklık (uygulanan) | Reasoning (uygulanan) | Ort. reasoning token | Sağlayıcılar (çağrı) | Çağrı |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| claude-haiku-4.5 | Anthropic | anthropic/claude-haiku-4.5 | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Amazon Bedrock (30) | 30 |
| claude-sonnet-4.6 | Anthropic | anthropic/claude-sonnet-4.6 | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Claude Platform on AWS (30) | 30 |
| deepseek-v3.2 | DeepSeek | deepseek/deepseek-v3.2 | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Baidu (13); DeepInfra (12); DigitalOcean (5) | 30 |
| gemini-3.1-flash-lite | Google | google/gemini-3.1-flash-lite | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Google (30) | 30 |
| glm-5 | Zhipu AI | z-ai/glm-5 | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Amazon Bedrock (15); Baidu (7); Z.AI (5); Novita (1); SiliconFlow (1); StreamLake (1) | 30 |
| gpt-4o | OpenAI | openai/gpt-4o | 0.6 (30) | 0.6 (30) | omitted (30) | 0.0 | OpenAI (30) | 30 |
| gpt-5.2 | OpenAI | openai/gpt-5.2 | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | OpenAI (30) | 30 |
| minimax-m2.5 | MiniMax | minimax/minimax-m2.5 | 0.6 (30) | 0.6 (30) | effort=minimal (30) | 691.6 | DigitalOcean (30) | 30 |
| qwen3.5-397b | Alibaba Cloud | qwen/qwen3.5-397b-a17b | 0.6 (30) | 0.6 (30) | effort=none (30) | 0.0 | Parasail (18); StreamLake (7); Alibaba (5) | 30 |

Notlar:

- gpt-4o için `reasoning` parametresi gönderilmedi (yapılandırmada kapalı; tabloda `omitted`).
- minimax-m2.5 için reasoning zorunludur. `param_tests.jsonl` dosyasında reasoning'i kapatmaya çalışan 3 istek HTTP 400 ile reddedildi (hata metni: "Reasoning is mandatory for this endpoint and cannot be disabled."). Ana deneyde en düşük düzey `effort=minimal` uygulandı.
- gpt-5.2 sıcaklık parametresini kabul eder ama uygulamaz. `param_tests.jsonl` dosyasında T = 0 ile yapılan 5 çağrı 5 farklı yanıt kümesi verdi; T = 1.5 ile yapılan 5 çağrı 5 farklı yanıt kümesi verdi.
- Aynı sınamanın claude-haiku-4.5 ve gpt-4o ile yapılan karşılaştırma çağrıları bu dosyaya kaydedilmedi; bu karşılaştırma yalnızca oturum içi bir kontroldür ve bu raporda sayısal kanıt olarak kullanılmamıştır.

### 2.2 Koşullar

Koşullar iki eksende tanımlanır: istem tekniği (sıfır atış = örnek çözüm ve akıl yürütme talimatı yok; CoT = adım adım düşünme talimatı) ve ağırlık yönlendirmesi (yönlendirmeli = istemde öncelik sırası ve önem ipuçları var; yönlendirmesiz = tarafsız sorular). Üç koşul çalıştırıldı; CoT × yönlendirmeli hücresi çalıştırılmadı. Kısa adlar: guided, direct (kayıt anahtarı `lean`), CoT (kayıt anahtarı `cot`).

| Teknik / yönlendirme | Yönlendirmeli | Yönlendirmesiz |
| --- | --- | --- |
| Sıfır atış | yönlendirmeli (guided): `guided` | yönlendirmesiz – doğrudan (direct): `lean` |
| CoT | çalıştırılmadı | yönlendirmesiz – CoT: `cot` |

| İstem içeriği | Yönlendirmeli (guided) | Yönlendirmesiz – doğrudan (direct) | Yönlendirmesiz – CoT |
| --- | --- | --- | --- |
| Rol satırı | tedarikçi değerlendirmesinde uzman AHP uzmanı, AHP kurallarıyla | tedarikçi değerlendirme deneyimli satın alma uzmanı | doğrudan ile aynı |
| Öncelik sırası | var: tipik öncelik sırası (Kalite en yüksek … Hacim düşük); “never use all 1s” | yok | yok |
| Ölçüt ipuçları | var: önem bildiren Türkçe ölçüt açıklamaları | yok (yalnız adlar) | yok (yalnız adlar) |
| KPI raporu | var: tedarikçilere ait Türkçe veri raporu | yok | yok |
| Akıl yürütme talimatı | her karşılaştırma için gerekçe istenir | yok | adım adım düşün, önce sıralamayı yaz |
| Yanıt biçimi | etiketli tam ikili karşılaştırma matrisi ve gerekçeler (JSON) | soru başına bir değer (JSON) | gerekçe, sıralama ve soru başına bir değer (JSON) |

Doğrudan ve CoT koşullarında model her soru için tek bir değer verir ve matrisi kod kurar; guided koşulunda model matrisi kendisi döndürür.

**yönlendirmesiz – doğrudan (direct).** Kayıtlarda bu koşulun anahtarı `lean`'dir (istem kimlikleri ve özetler bu anahtarla tutulur). Sistem mesajı:

```
You are a procurement professional with experience in supplier evaluation.
Answer the following questions. Respond only in the requested JSON format.
```

Kullanıcı mesajı; ölçütlerin kısa tanıtımını, 17 düzeyli Saaty ölçeğini (9 … 1 … 1/9), soru listesini ve yalnızca `answers` alanını isteyen JSON biçimini içerir. Ölçütler için tanım, öncelik ipucu ya da veri verilmez.

**Yönlendirmesiz – CoT (adım adım düşünme).** CoT de ağırlıklar açısından yönlendirmesizdir: doğrudan isteme yalnız adım adım düşünme talimatı ekler (kayıtlarda anahtarı `cot`). Sistem mesajı, ölçek ve sorular yönlendirmesiz – doğrudan (direct) koşulla aynıdır. Yanıt biçiminin yerine şu metin gelir:

```
Before answering, think step by step:
1. State in one sentence what each criterion represents in supplier evaluation.
2. Rank the criteria from most to least important in your view.
3. Choose each value so that it is consistent with this ranking
   (for example, if A is more important than B and B than C, A must be even more important than C).

Response format (only this JSON):
{"reasoning": "your brief reasoning for steps 1-3",
 "ranking": ["most important criterion", "...", "least important criterion"],
 "answers": [{"question": 1, "value": "3"}, ...]}
```

**Yönlendirmeli (guided).** Tezde kullanılan özgün istem `prompts/guided/` dosyalarından bayt bayt aynen gönderilir (denetim d). Sistem mesajı AHP kurallarını ve “never use all 1s” kuralını içerir. Ana ölçüt kullanıcı mesajı bir JSON nesnesidir; üst düzey alanları: `action`, `scenario_context`, `main_criteria`, `instructions`, `output_schema`. `instructions` alanının alt alanları: `task`, `priority_guidance`, `differentiation`, `matrix_rules`. Bu istem bir öncelik sırası (`priority_guidance`), ipucu taşıyan ölçüt açıklamaları (`main_criteria[].description`) ve `scenario_context` içinde Türkçe tedarikçi KPI raporunu içerir. Teslimat alt ölçüt mesajının alanları: `action`, `scenario_context`, `main_criterion`, `sub_criteria`, `instructions`, `output_schema`.

Ana deneydeki ortalama istem (prompt) token sayısı:

| Koşul | Görev | Çağrı | Ort. prompt token |
| --- | --- | --- | --- |
| direct | main | 45 | 536.5 |
| direct | delivery | 45 | 479.0 |
| CoT | main | 45 | 631.9 |
| CoT | delivery | 45 | 574.5 |
| guided | main | 45 | 4884.0 |
| guided | delivery | 45 | 4709.9 |

### 2.3 Görevler

- `main`: 5 ana ölçüt, 10 soru: Delivery and Operational Performance; Financial Performance; Quality Performance; Volume and Logistics; Compliance and Reliability.
- `delivery`: Teslimat ve operasyonel performansın 4 alt ölçütü, 6 soru: Delivered Orders; Pending Orders; Cancelled Orders; Partially Delivered Orders.
- Soru sırası uzmanlara sorulan sorulardaki sıradır: (1,2), (1,3), …, (n−1,n).

### 2.4 Tekrarlar ve sıcaklık kolları

- Ana deney: her model × koşul × görev hücresi için 5 bağımsız çağrı, sıcaklık 0.6.
- `temp_0.0`: yalnız yönlendirmesiz – doğrudan (direct) koşul, 3 tekrar, sıcaklık 0.0; 54 çağrı.
- `temp_1.0`: yalnız yönlendirmesiz – doğrudan (direct) koşul, 3 tekrar, sıcaklık 1.0; 54 çağrı.
- `pilot` etiketli 10 çağrı ön denemedir ve sonuçlara alınmadı.

## 3. Kayıt (loglama) ve doğrulanabilirlik

### 3.1 Ne kaydedildi, nerede

Her API çağrısı tek bir JSON satırı olarak eklenir; dosyalar yalnızca sona ekleme ile büyür.

- `results/raw/calls.jsonl`: ana deney, sıcaklık kolları ve pilot çağrıları.
- `results/raw/revisions.jsonl`: revizyon aşamasındaki panel modeli çağrıları.
- `results/raw/judge_calls.jsonl`: hakem modeli çağrıları (geri bildirim ve karar).
- `results/raw/param_tests.jsonl`: parametre sınamaları (gpt-5.2 sıcaklığı, minimax-m2.5 reasoning).
- `results/*.log`: çalıştırma sırasında ekrana yazılan ilerleme satırları.

Her çağrı kaydındaki alanlar: gönderilen mesajların tamamı (`messages`), istem kimliği ve SHA-256 özeti (`prompt_id`, `prompt_sha256`), istenen ve uygulanan parametreler (`requested_params`, `applied_params`, atlanan parametreler `omitted`), her HTTP denemesi (`http_attempts`), ham yanıt (`raw_reply`), modelin akıl yürütme metni varsa (`model_reasoning_text`), bitiş nedeni, token kullanımı (prompt, completion, reasoning), maliyet, sağlayıcı, OpenRouter üretim kimliği (`generation_id`), gecikme ve zaman damgası. Ayrıştırma sonucunda yanıtlar, matris, ağırlıklar, λmax, CI ve CR aynı kayda yazılır. API anahtarı kayda girmeden önce maskelenir (`client._scrub`).

### 3.2 Denetim sonuçları

`python -m ahp_bench.audit` 86 beklenti denetimi yaptı; 86 tanesi geçti, 0 tanesi kaldı. Ayrıntılar `results/report/audit/audit.md` ve aynı klasördeki CSV dosyalarındadır.

**a. Envanter.** Dosya ve etiket başına kayıt, başarı, ayrıştırma, maliyet ve token sayıları:

| Dosya | run_tag | Kayıt | Başarılı çağrı | Başarısız | parse_ok | Maliyet (USD) | Prompt tok. | Completion tok. | Reasoning tok. |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| calls.jsonl | main | 270 | 270 | 0 | 270 | 1.4783 | 531710 | 151942 | 20749 |
| calls.jsonl | pilot | 10 | 9 | 1 | 9 | 0.0142 | 4828 | 2185 | 924 |
| calls.jsonl | temp_0.0 | 54 | 54 | 0 | 54 | 0.0845 | 27420 | 11481 | 4213 |
| calls.jsonl | temp_1.0 | 54 | 54 | 0 | 54 | 0.0948 | 27422 | 15986 | 8453 |
| revisions.jsonl | rev_final | 7 | 7 | 0 | 7 | 0.0078 | 7359 | 892 | 0 |
| revisions.jsonl | rev_test_none | 2 | 2 | 0 | 2 | 0.0005 | 1881 | 188 | 0 |
| revisions.jsonl | revision_final | 250 | 250 | 0 | 250 | 0.6633 | 283623 | 103386 | 42074 |
| revisions.jsonl | revision_judge | 245 | 245 | 0 | 227 | 0.6164 | 393378 | 80440 | 27826 |
| revisions.jsonl | revision_v2 | 6 | 6 | 0 | 6 | 0.0186 | 5440 | 498 | 0 |
| judge_calls.jsonl | rev_final | 14 | 14 | 0 | 14 | 0.1479 | 25512 | 20346 | 18583 |
| judge_calls.jsonl | rev_test_none | 4 | 4 | 0 | 4 | 0.0351 | 6912 | 4698 | 4226 |
| judge_calls.jsonl | revision_final | 500 | 500 | 0 | 500 | 0.4735 | 387505 | 48817 | 0 |
| judge_calls.jsonl | revision_judge | 458 | 458 | 0 | 458 | 0.4240 | 337460 | 43236 | 0 |
| judge_calls.jsonl | revision_v2 | 12 | 12 | 0 | 12 | 0.1829 | 21002 | 27199 | 25539 |
| param_tests.jsonl | (param test) | 14 | 11 | 3 |  | 0.0241 | 0 | 0 | 720 |

**b. Tasarımın tamlığı.** `main` etiketinde beklenen 270 hücrenin 270 tanesi kayıtlıdır; eksik 0, yinelenen 0, beklenmeyen 0 hücre vardır; ayrıştırılmış kaydı olmayan hücre sayısı: 0. `temp_0.0` için beklenen 54, kayıtlı 54, eksik 0, yinelenen 0. `temp_1.0` için beklenen 54, kayıtlı 54, eksik 0, yinelenen 0.

**c. Alan tamlığı.** Sonuç etiketlerindeki her kayıtta alanın dolu olduğu kayıt sayısı (`usage_tokens`: prompt ve completion token tamsayı; hakem kayıtlarında istem kimliği tutulmaz, “—”):

| Dosya | run_tag | Kayıt | messages | raw_reply | requested_params | applied_params | usage_tokens | cost | provider | generation_id | timestamp_utc | prompt_id | prompt_sha256 | Eksik alanlı kayıt |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| calls.jsonl | main | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 270 | 0 |
| calls.jsonl | temp_0.0 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 0 |
| calls.jsonl | temp_1.0 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 54 | 0 |
| revisions.jsonl | revision_final | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 250 | 0 |
| judge_calls.jsonl | revision_final | 500 | 500 | 500 | 500 | 500 | 500 | 500 | 500 | 500 | 500 | — | — | 0 |

**d. İstem bütünlüğü.** `main` etiketinde 6 koşul × görev çiftinin her birinde farklı `prompt_sha256` sayısı: 1, 1, 1, 1, 1, 1. Saklanan özet ile saklanan mesajların yeniden hesaplanan özeti 270/270 kayıtta aynıdır; saklanan mesajlar bugünkü kodun ürettiği mesajlarla 270/270 kayıtta aynıdır. guided/main: sistem mesajı dosyayla (CRLF→LF) 45/45, kullanıcı mesajı dosya baytlarıyla 45/45 kayıtta aynı. guided/delivery: sistem mesajı dosyayla (CRLF→LF) 45/45, kullanıcı mesajı dosya baytlarıyla 45/45 kayıtta aynı. Revizyon kayıtlarında saklanan özet ile mesaj özeti 250/250 kayıtta aynıdır.

**e. Uygulanan ayarlar.** Ana deneyde istenen ve uygulanan sıcaklık her modelde aynıdır (2.1'deki tablo). Parametre düşürme (reasoning ya da sıcaklığın geri çekilmesi) gerektiren çağrı sayısı: 0. 3 model birden fazla sağlayıcıya yönlendirildi: deepseek-v3.2: Baidu (13); DeepInfra (12); DigitalOcean (5); glm-5: Amazon Bedrock (15); Baidu (7); Z.AI (5); Novita (1); SiliconFlow (1); StreamLake (1); qwen3.5-397b: Parasail (18); StreamLake (7); Alibaba (5). Ana deney çağrılarının bitiş nedenleri: stop (270).

**f. Ayrıştırma.** Ayrıştırıcı ana deneyden sonra iyileştirildi ve saklanan ham yanıtlar yeniden ayrıştırıldı (`parser_version`). Kayıtların ayrıştırıcı sürümüne göre dağılımı (`absent`: alan yok):

| run_tag | parser_version | records |
| --- | --- | --- |
| main | 3 | 162 |
| main | absent | 108 |
| pilot | 3 | 9 |
| pilot | absent | 1 |
| temp_0.0 | absent | 54 |
| temp_1.0 | absent | 54 |

`main` etiketinde `parser_version` alanı olmayan 108 kaydın ilk zaman damgası 2026-11-23T18:07:24+00:00; yeniden ayrıştırma öncesi yedekteki son `main` kaydının zaman damgası 2026-11-23T17:26:09+00:00. Bu kayıtlar yeniden ayrıştırmadan sonra yapılan çağrılardır ve çağrı anında ayrıştırılmıştır; g bölümündeki yeniden ayrıştırma bunlar için de aynı sonucu verir.

Guided yanıtlarında etiket ve karşılıklılık durumu (ana deney):

| task | records | parse_ok | labels_verified | labels_not_verified | reordered | reciprocal_as_returned | not_reciprocal_as_returned | upper_on_scale | bare_fractions_quoted |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| main | 45 | 45 | 43 | 2 | 2 | 40 | 5 | 40 | 13 |
| delivery | 45 | 45 | 43 | 2 | 0 | 41 | 4 | 44 | 6 |

`labels_verified`: döndürülen etiketler ölçütlerle bire bir eşlendi; `reordered`: model ölçütleri farklı sırada verdi ve matris kanonik sıraya çevrildi; `reciprocal_as_returned`: döndürülen matris köşegeni 1 ve a_ij·a_ji = 1 (±0.01); analizde her durumda üst üçgenden karşılıklı matris kurulur; `bare_fractions_quoted`: tırnaksız kesirler (1/3) JSON için tırnağa alındı. Yönlendirmesiz – doğrudan (direct) ve CoT yanıtlarında bu işlem 0 kez gerekti.

Yedek dosyalar (`calls_backup_before_reparse.jsonl`, `calls.jsonl.bak`) üretim kimliğiyle eşlendi; ham alanlar (ham yanıt, mesajlar, kullanım, maliyet, sağlayıcı, zaman, parametreler) calls_backup_before_reparse.jsonl: 172/172 kayıtta aynı; calls.jsonl.bak: 172/172 kayıtta aynı. Yani ham yanıtlar yeniden ayrıştırmada değişmedi. Bu kayıtlarda ayrıştırılan kayıt sayısı yedekte 153 (calls_backup_before_reparse.jsonl), 161 (calls.jsonl.bak); aynı kayıtlarda bugün: 171.

**g. Yeniden hesaplama.** Her ayrıştırılmış kayıt için matris saklanan yanıtlardan (direct, CoT) ya da döndürülen matristen (guided) yeniden kuruldu; ağırlık, λmax, CI ve CR `ahp` fonksiyonlarıyla yeniden hesaplandı. Saklanan değerlerle en büyük mutlak fark: 1.51e-14.

| Dosya | run_tag | Kayıt | Matris | Ağırlık | λmax | CI | CR |
| --- | --- | --- | --- | --- | --- | --- | --- |
| calls.jsonl | main | 270 | 0 | 4.44e-16 | 1.15e-14 | 3.28e-15 | 3.64e-15 |
| calls.jsonl | pilot | 9 | 0 | 2.22e-16 | 8.88e-15 | 2.22e-15 | 1.98e-15 |
| calls.jsonl | temp_0.0 | 54 | 0 | 5.55e-16 | 1.24e-14 | 4.00e-15 | 4.44e-15 |
| calls.jsonl | temp_1.0 | 54 | 0 | 3.89e-16 | 1.24e-14 | 3.11e-15 | 3.28e-15 |
| revisions.jsonl | rev_final | 7 | 0 | 2.78e-16 | 3.55e-15 | 1.18e-15 | 1.30e-15 |
| revisions.jsonl | rev_test_none | 2 | 0 | 2.78e-17 | 5.33e-15 | 1.78e-15 | 1.97e-15 |
| revisions.jsonl | revision_final | 250 | 0 | 4.16e-16 | 1.15e-14 | 3.55e-15 | 3.94e-15 |
| revisions.jsonl | revision_judge | 227 | 0 | 3.61e-16 | 1.51e-14 | 5.00e-15 | 5.55e-15 |
| revisions.jsonl | revision_v2 | 6 | 0 | 6.66e-16 | 4.44e-15 | 1.48e-15 | 1.65e-15 |

Ayrıca tüm ham yanıtlar bugünkü ayrıştırıcıyla dosyaya yazmadan yeniden ayrıştırıldı: calls.jsonl main: parse_ok uyumu 270/270, bayrak farkı 0, en büyük CR farkı 3.64e-15; calls.jsonl temp_0.0: parse_ok uyumu 54/54, bayrak farkı 0, en büyük CR farkı 4.44e-15; calls.jsonl temp_1.0: parse_ok uyumu 54/54, bayrak farkı 0, en büyük CR farkı 3.28e-15; revisions.jsonl revision_final: parse_ok uyumu 250/250, bayrak farkı 0, en büyük CR farkı 3.94e-15.

**h. Gizli bilgi taraması.** `results/`, `data/`, `prompts/` altındaki 75 dosya (`__pycache__` hariç) OpenRouter anahtar öneki ve `Bearer` sözcüğü için tarandı. Anahtar biçimindeki eşleşme sayısı: 0; `Bearer` sözcüğünden sonra belirteç biçiminde dize sayısı: 0. Anahtar önekini içeren dosya sayısı: 0. `Bearer` sözcüğünü içeren dosyalar (içerik yazdırılmadı): yok. `.env` dosyası taranan klasörlerin dışındadır ve okunmadı.

**i. Sağlama toplamları.** `results/raw` dosyalarının boyutu ve SHA-256 özeti:

| Dosya | Bayt | Satır | SHA-256 |
| --- | --- | --- | --- |
| calls.jsonl | 4327321 | 388 | 20b09aa92084d3da424096685692c3e9cce5a6f3f538f0aa7e50427ae0b4f2a8 |
| calls.jsonl.bak | 2216338 | 172 | 07825055ca382840ba9ec0dbd52b30813c7cc89b7d502825eef76e7a52d4f638 |
| calls_backup_before_reparse.jsonl | 2192793 | 172 | 3c0b195a396c04e403388d56c402e80db044db38ccc1791f7ab204c55581f756 |
| judge_calls.jsonl | 5080243 | 988 | 3597f1edf7e9983afc796ca877bca2b4ac7390c1d45fe559f3ba2ce7cc02e741 |
| param_tests.jsonl | 7089 | 14 | 9f4dc24d9885ea19d574ab0c0ee67cd46511ccd5a715520c98391bef435beb47 |
| revisions.jsonl | 5938748 | 510 | 55128d02a535d4378961b57d582efe8bf77c7cbc373494f80650217eb1a9c549 |

### 3.3 Nasıl kontrol edilir

Kod `https://github.com/baa-git/llm-ahp-mcdm` deposundadır (sürüm v0.2.0); kayıtlar, veriler ve bu rapor `https://github.com/baa-git/llm-ahp-mcdm-study` deposundadır. Komutlar kod deposunun kökünde çalıştırılır; `<study>` çalışma deposunun yerel klasörüdür. Hiçbiri API çağrısı yapmaz.

1. `python -m unittest discover -s tests -v` — Çevrimdışı birim testleri (sahte HTTP katmanı ve yapay veri). Ayrıştırıcının, AHP hesaplarının, revizyon protokolünün, raporun ve denetimin beklenen davranışı verdiğini gösterir.
2. `python -m ahp_bench.audit --results <study>/results` — Bu bölümdeki a–j denetimlerini saklanan dosyalar üzerinde yeniden yapar; `results/raw` dosyalarını yalnızca okur. Tamlığı, istem bütünlüğünü, yeniden hesaplamayı, gizli bilgi yokluğunu ve revizyon kurallarına uyumu gösterir.
3. `python -m ahp_bench.report --results <study>/results --data <study>/data --tag main --revision-tag revision_final` — Tüm tabloları, şekilleri ve ekleri kayıtlardan yeniden üretir. Çıktı belirlenimlidir (aynı girdiyle aynı tablolar).
4. `python -m ahp_bench.tech_report --results <study>/results --data <study>/data` — Bu raporu yeniden üretir.
5. `python -m ahp_bench.reparse --results <study>/results` — Ham yanıtları bugünkü ayrıştırıcıyla yeniden ayrıştırır ve `calls.jsonl` içindeki türetilmiş alanları yeniden yazar (önce `.bak` kopyası alır; ham alanlara dokunmaz). Dosyayı değiştirdiği için yalnız doğrulama amacıyla gerekmez: denetimin g bölümü aynı karşılaştırmayı dosyaya yazmadan yapar.
6. Sağlama toplamı: `certutil -hashfile results\raw\calls.jsonl SHA256` (Windows) ya da `sha256sum results/raw/*` çıktısı 3.2-i tablosuyla karşılaştırılır. Aynı özet, dosyanın bu rapordan sonra değişmediğini gösterir.

## 4. Hesaplama kuralları

- **Yanıtlardan matris.** k. soru (i, j) çiftine aittir. Değer v ise a_ij = v, a_ji = 1/v, a_ii = 1. Guided koşulunda model matrisi döndürür; etiketler kanonik sıraya eşlenir ve yalnız üst üçgen kullanılır (a_ji = 1/a_ij).
- **Ağırlıklar.** A·w = λmax·w denkleminin en büyük gerçel özdeğerine ait sağ özvektör; |w| alınır ve Σw = 1 olacak biçimde normalleştirilir (`numpy.linalg.eig`).
- **Tutarlılık.** CI = (λmax − n)/(n − 1); CR = CI/RI; RI = n = 3: 0.58, n = 4: 0.9, n = 5: 1.12. Matris, CR dört ondalığa yuvarlandığında 0.10 veya altındaysa tutarlı sayılır (`round(CR, 4) ≤ 0.10`).
- **Grup matrisi.** Her hücre için geometrik ortalama: g_ij = (Π_k a_ij^(k))^(1/K). Grup ağırlıkları bu matrisin özvektörüdür. Uzman grubu K = 7, bir koşulun LLM paneli K = 9 model × 5 tekrar = 45 matristir; model düzeyinde K = 5 tekrardır.
- **Global ağırlıklar.** Teslimat alt ölçütlerinin yerel ağırlıkları Teslimat ana ağırlığıyla çarpılır; diğer dört ana ölçüt ağırlığı aynen alınır: (D1…D4) = w_Teslimat · (d1…d4); F, Q, V, C = ana ağırlıklar. Sekiz global ağırlığın toplamı 1'dir.
- **Teslimat göstergelerinin orana çevrilmesi.** D1–D4 sipariş sayılarıdır. Her tedarikçi için D_j' = 100 · D_j / (D1 + D2 + D3 + D4). Ana tablolar (T11, T13–T15) oranlarla hesaplanır; T12 sayılarla ve oranlarla karşılaştırır. Gösterge yönleri (`performance_matrix.json`): D1 (fayda), D2 (maliyet), D3 (maliyet), D4 (maliyet), F (fayda), Q (maliyet), V (fayda), C (maliyet).
- **TOPSIS.** r_ij = x_ij / √(Σ_i x_ij²); v_ij = w_j · r_ij. İdeal çözüm fayda göstergesinde en büyük, maliyet göstergesinde en küçük v_ij; ideal olmayan çözüm tersidir. D_i+ ve D_i− Öklid uzaklıklarıdır; puan C_i = D_i− / (D_i+ + D_i−).
- **ARAS (η = 1).** Maliyet göstergeleri 1/x ile çevrilir. İdeal satır x_0j = max_i x_ij eklenir; normalleştirme x̄_ij = x_ij / (Σ_i x_ij + x_0j). S_i = Σ_j w_j x̄_ij, puan K_i = S_i / S_0. Kodda ek bir üs ya da katsayı yoktur.
- **EDAS.** AV_j = sütun ortalaması. Fayda göstergesinde PDA = max(0, (x − AV)/AV), NDA = max(0, (AV − x)/AV); maliyet göstergesinde tersi. SP_i = Σ w_j PDA_ij, SN_i = Σ w_j NDA_ij; NSP = SP/max SP, NSN = 1 − SN/max SN; puan AS_i = (NSP + NSN)/2.
- **Sıralar.** Büyük puan 1. sıradır; eşitlikte ortalama sıra verilir.
- **Benzerlik.** Ağırlık vektörleri arasında Spearman ρ, Kendall τ-b, RMSE ve ortalama mutlak fark; son ikisi yüzde puan (ağırlık × 100) cinsindendir.

## 5. Orkestrasyon ve revizyon

### 5.1 Akış

Revizyon yalnız yönlendirmesiz – doğrudan (direct) ve CoT koşullarında, ana deneyin ilk yanıtlarından CR > 0.10 olan matrisler için yapıldı. Hakem model: `x-ai/grok-4.20` (sıcaklık 0.0, reasoning parametresi gönderilmedi). Her zincir şu adımlarla ilerler:

1. Kod, o ana kadarki en iyi geçerli yanıtları seçer: ilk yanıt ve ayrıştırılan tüm revizyonlar içinde CR'si en düşük olan (eşitlikte önceki).
2. Kod bu yanıtların CR'sini, en tutarsız üç üçlüyü (|ln a_ij + ln a_jk − ln a_ik| büyükten küçüğe) ve en çok üç döngüsel (sıralı) üçlüyü hesaplar.
3. Hakem model bu listeyi, yanıtları ve CR'yi alır; en çok 120 sözcüklük bir geri bildirim yazar. Değer önermesi ve hangi ölçütün daha önemli olduğunu söylemesi yasaktır.
4. Panel modeline şu konuşma gönderilir: özgün mesajlar + tek bir asistan turu (özgün yanıt ya da en iyi yanıtlar, koşulun JSON biçiminde) + yeni kullanıcı mesajı. Yeni mesaj hakemin geri bildirimi ve kodun eklediği sabit kuyruktan oluşur. Hakem geri bildirimi okunamazsa kodun yazdığı revizyon istemi kullanılır.
5. Kod yanıtı ayrıştırır ve CR'yi hesaplar. Yanıt ayrıştırılabildiyse hakem, ilk yanıtlarla revize yanıtları karşılaştırıp `logic_preserved` = yes / partly / no kararı ve kısa bir yorum verir.
6. Durma kuralı: `round(CR, 4) ≤ 0.10` olan ilk revizyon ya da 5 revizyon. Zincirin son yanıtları en iyi geçerli yanıtlardır (en düşük CR). Ayrıştırılamayan yanıt bir revizyon hakkı sayılır ve aynı konuşma yeniden gönderilir.

Kodun hakem geri bildiriminin sonuna eklediği kuyruk (`{previous_answers}` yerine en iyi yanıtlar `Q1: Left … | Right … = değer` biçiminde yazılır):

```
Your previous answers:
{previous_answers}

Use only these values: 1/9, 1/8, 1/7, 1/6, 1/5, 1/4, 1/3, 1/2, 1, 2, 3, 4, 5, 6, 7, 8, 9.
You may change the direction of an answer if that is what resolves a contradiction.
Answer all questions again in the same JSON format.
```

Hakem geri bildirimi kullanılamazsa gönderilen kod istemi:

```
The comparison matrix built from your answers has a consistency ratio CR = {cr}.
The accepted limit is 0.10; your answers contradict each other.

Most contradictory comparisons:
{contradictions}

Your previous answers:
{previous_answers}

Review your answers. You do not have to change your view; make only the changes needed
to remove the contradiction.
Use only these values: 1/9, 1/8, 1/7, 1/6, 1/5, 1/4, 1/3, 1/2, 1, 2, 3, 4, 5, 6, 7, 8, 9.
You may change the direction of an answer if that is what resolves a contradiction.
Answer all questions again in the same JSON format.
```

### 5.2 Kim neye karar verir

| Adım | Karar veren |
| --- | --- |
| CR, tutarsız ve döngüsel üçlüler | kod |
| Geri bildirim metni | hakem model |
| Önceki yanıtların listesi, ölçek ve biçim talimatı | kod (sabit kuyruk) |
| Revize değerler | panel modeli |
| Revize CR ve tutarlılık | kod |
| Mantık korunumu kararı (yes/partly/no) | hakem model (yalnız kayıt; durmayı etkilemez) |
| Durma ve en iyi yanıtın seçimi | kod |

Kayıt: her panel çağrısı `revisions.jsonl` dosyasına (gönderilen konuşma, dayanılan yanıtlar ve CR, hakem geri bildirimi, hakem kararı, mantık ölçüleri, maliyetler), her hakem çağrısı `judge_calls.jsonl` dosyasına (amaç `feedback` ya da `verdict`, mesajlar, ham yanıt, ayrıştırma sonucu) yazılır.

### 5.3 Revizyon denetimi ve sonuçları

`revision_final` etiketinde: revizyon gereken matris 89, kaydı olan zincir 89; panel çağrısı 250 (başarılı 250, ayrıştırılan 250); hakem çağrısı 500 (feedback 250, verdict 250; başarılı 500, ayrıştırılan 500). Hakem geri bildirimi okunamadığı için kod istemine düşülen çağrı 0, yeniden kullanılan geri bildirim 0; geçerli karar 250, geçersiz karar 0, karar istenmeyen çağrı 0. Tüm çağrılardaki kararlar: yes 45, partly 81, no 124. Tutarlılığa ulaşan zincir 69. Kural ihlali (numara sırası, durma kuralı, en iyi yanıt kuralı, konuşma yapısı, kuyruk metni, hakem çağrısı sayısı) 0. Hakem modeli kimliği: x-ai/grok-4.20.

T16 panel satırları (toplam revizyon gereken 89, 5 revizyon içinde tutarlı hale gelen 69, ulaşmayan 20):

| Condition | Task | Matrices | Needing revision | Reached after ≤1 | Reached after ≤2 | Reached after ≤3 | Reached after ≤4 | Reached after ≤5 | Not reached | Mean initial CR | Mean final CR |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| direct | main | 45 | 30 | 8 | 12 | 14 | 17 | 19 | 11 | 0.2884 | 0.0971 |
| direct | delivery | 45 | 22 | 5 | 13 | 14 | 15 | 17 | 5 | 0.5356 | 0.0907 |
| CoT | main | 45 | 21 | 7 | 10 | 13 | 14 | 17 | 4 | 0.3820 | 0.0751 |
| CoT | delivery | 45 | 16 | 11 | 13 | 14 | 15 | 16 | 0 | 0.1726 | 0.0443 |

T17 ilk ve son yanıtlarla panel ana ölçüt ağırlıkları ve uzmanlarla benzerlik:

| Condition | Answers | Main matrices | Delivery | Financial | Quality | Volume | Compliance | Spearman rho main | RMSE main (pp) | Spearman rho global | RMSE global (pp) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Experts |  | 7 | 0.2147 | 0.1604 | 0.3186 | 0.0529 | 0.2535 |  |  |  |  |
| direct | first | 45 | 0.1744 | 0.0796 | 0.3507 | 0.0870 | 0.3083 | 0.900 | 5.17 | 0.429 | 4.47 |
| direct | final | 45 | 0.1548 | 0.0849 | 0.3442 | 0.1001 | 0.3160 | 0.900 | 5.67 | 0.500 | 4.53 |
| CoT | first | 45 | 0.1864 | 0.0950 | 0.3757 | 0.0800 | 0.2629 | 1.000 | 4.28 | 0.548 | 4.22 |
| CoT | final | 45 | 0.1914 | 0.0842 | 0.4165 | 0.0641 | 0.2438 | 1.000 | 5.69 | 0.571 | 5.20 |

T20 panel satırları (son yanıtlar ilk yanıtlarla karşılaştırılır; hakem kararları son yanıtları veren revizyona aittir):

| Condition | Task | Chains revised | Reached CR ≤ 0.10 | Mean revisions used | Mean answers changed | Mean Spearman rho | Top criterion unchanged | Ranking unchanged | Mean max abs weight change (pp) | Judge yes | Judge partly | Judge no |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| direct | main | 30 | 19 | 3.300 | 4.233 | 0.813 | 0.8000 | 0.4000 | 9.06 | 7 | 13 | 8 |
| direct | delivery | 22 | 17 | 2.864 | 3.955 | 0.626 | 0.6364 | 0.1818 | 16.46 | 1 | 6 | 15 |
| CoT | main | 21 | 17 | 2.905 | 5.333 | 0.714 | 0.6190 | 0.1905 | 15.86 | 7 | 3 | 11 |
| CoT | delivery | 16 | 16 | 1.688 | 4.250 | 0.675 | 0.9375 | 0.3750 | 10.91 | 8 | 2 | 6 |

### 5.4 Örnekler (A08'den, dosya sırasındaki ilk “yes” ve ilk “no” kararı)

**claude-haiku-4.5, CoT, delivery, tekrar 2, revizyon 1** (CR 0.1971 → 0.1682, karar: yes)

> Geri bildirim: Your answers show contradictions. For instance, Pending vs Cancelled (1/5), Cancelled vs Partially (1/3), and Pending vs Partially (1/2) form a circular preference; at least one of those three must move toward the opposite direction. Similarly, Delivered vs Pending (5), Pending vs Cancelled (1/5), and Delivered vs Cancelled (7) are inconsistent, as are Delivered vs Cancelled (7), Cancelled vs Partially (1/3), and Delivered vs Partially (3). Please reconsider only these comparisons (Q1,Q2,Q3,Q4,Q5,Q6), adjusting solely what is needed to resolve the inconsistencies while preserving your ranking: Delivered > Partially > Pending > Cancelled.
>
> Hakem yorumu: Revised answers strengthen original view that Delivered Orders dominate even more, while keeping the same overall ranking and relative ordering of the other three criteria. Slight tightening of ratios improves consistency without reversing any priorities.

**claude-haiku-4.5, CoT, main, tekrar 3, revizyon 2** (CR 0.1235 → 0.3574, karar: no)

> Geri bildirim: Your answers show inconsistencies. Specifically: Delivery vs Financial (3), Financial vs Quality (1/3), but Delivery vs Quality (1/5) form a contradiction. Also, Financial vs Quality (1/3), Quality vs Volume (5), but Financial vs Volume (1/3) conflict. And Financial vs Quality (1/3), Quality vs Compliance (3), but Financial vs Compliance (1/5) are inconsistent. These three triplets contradict your own ranking. Please reconsider only those answers, adjusting them toward consistency while keeping your view of what matters most.
>
> Hakem yorumu: Original ranking placed Compliance and Quality far above Delivery/Operational. Revised answers invert this, elevating Delivery to top rank while demoting Quality and altering relative gaps, indicating priorities were changed to force consistency rather than preserving the original view.

## 6. Bulgular

Bu bölüm `results/report/report.md` tablolarını özetler. Tablolar ilk yanıtlara (`main`) dayanır; T16, T17 ve T20 `revision_final` kayıtlarını kullanır.

### T01 Modeller ve ayarlar

Model başına çağrı sayısı, sağlayıcılar, uygulanan reasoning ve maliyet. Toplam çağrı 270, ayrıştırma hatası 0, toplam maliyet 1.4783 USD (ayrıntı 2.1).

### T02 Uzman tutarlılığı

7 uzmanın bireysel CR değerleri. CR ≤ 0.10 olan uzman matrisi: ana ölçütlerde 4/7, teslimat alt ölçütlerinde 0/7. Geometrik ortalama grup matrisinin CR'si ana ölçütlerde 0.0321, teslimatta 0.0208. Bireysel CR aralığı ana ölçütlerde 0.0142–0.3932, teslimatta 0.1140–0.4938.

### T03 LLM tutarlılığı

Model ve koşul başına ortalama ve en büyük CR ile CR ≤ 0.10 olan matris sayısı; panel satırları tüm modelleri birleştirir ve grup matrisinin CR'sini verir. direct: ana görevde 15/45, teslimatta 23/45 matris tutarlı; grup CR 0.0616 / 0.0866. CoT: ana görevde 24/45, teslimatta 29/45 matris tutarlı; grup CR 0.0228 / 0.0075. guided: ana görevde 43/45, teslimatta 45/45 matris tutarlı; grup CR 0.0315 / 0.0033.

| Model | Condition | Mean CR main | CR ≤ 0.10 main | Group CR main | Mean CR delivery | CR ≤ 0.10 delivery | Group CR delivery |
| --- | --- | --- | --- | --- | --- | --- | --- |
| claude-haiku-4.5 | direct | 0.1576 | 0/5 |  | 0.0951 | 4/5 |  |
| claude-haiku-4.5 | CoT | 0.1150 | 2/5 |  | 0.0962 | 3/5 |  |
| claude-haiku-4.5 | guided | 0.0272 | 5/5 |  | 0.0233 | 5/5 |  |
| claude-sonnet-4.6 | direct | 0.0418 | 5/5 |  | 0.0753 | 5/5 |  |
| claude-sonnet-4.6 | CoT | 0.0682 | 4/5 |  | 0.0489 | 5/5 |  |
| claude-sonnet-4.6 | guided | 0.0284 | 5/5 |  | 0.0331 | 5/5 |  |
| deepseek-v3.2 | direct | 0.2683 | 0/5 |  | 0.1298 | 3/5 |  |
| deepseek-v3.2 | CoT | 0.0871 | 2/5 |  | 0.1122 | 4/5 |  |
| deepseek-v3.2 | guided | 0.0585 | 5/5 |  | 0.0304 | 5/5 |  |
| gemini-3.1-flash-lite | direct | 0.0631 | 5/5 |  | 0.9659 | 0/5 |  |
| gemini-3.1-flash-lite | CoT | 0.1037 | 3/5 |  | 0.0495 | 4/5 |  |
| gemini-3.1-flash-lite | guided | 0.0280 | 5/5 |  | 0.0093 | 5/5 |  |
| glm-5 | direct | 0.1391 | 2/5 |  | 0.1210 | 4/5 |  |
| glm-5 | CoT | 0.1342 | 3/5 |  | 0.0800 | 4/5 |  |
| glm-5 | guided | 0.0249 | 5/5 |  | 0.0193 | 5/5 |  |
| gpt-4o | direct | 0.4381 | 0/5 |  | 0.6789 | 0/5 |  |
| gpt-4o | CoT | 0.2681 | 3/5 |  | 0.2018 | 0/5 |  |
| gpt-4o | guided | 0.0398 | 5/5 |  | 0.0256 | 5/5 |  |
| gpt-5.2 | direct | 0.0917 | 3/5 |  | 0.1258 | 3/5 |  |
| gpt-5.2 | CoT | 0.0983 | 4/5 |  | 0.1055 | 3/5 |  |
| gpt-5.2 | guided | 0.0974 | 3/5 |  | 0.0189 | 5/5 |  |
| minimax-m2.5 | direct | 0.4071 | 0/5 |  | 0.3180 | 2/5 |  |
| minimax-m2.5 | CoT | 0.1854 | 3/5 |  | 0.0526 | 5/5 |  |
| minimax-m2.5 | guided | 0.0394 | 5/5 |  | 0.0351 | 5/5 |  |
| qwen3.5-397b | direct | 0.2967 | 0/5 |  | 0.1612 | 2/5 |  |
| qwen3.5-397b | CoT | 0.8310 | 0/5 |  | 0.1245 | 1/5 |  |
| qwen3.5-397b | guided | 0.0478 | 5/5 |  | 0.0319 | 5/5 |  |
| Panel | direct | 0.2115 | 15/45 | 0.0616 | 0.2968 | 23/45 | 0.0866 |
| Panel | CoT | 0.2101 | 24/45 | 0.0228 | 0.0968 | 29/45 | 0.0075 |
| Panel | guided | 0.0435 | 43/45 | 0.0315 | 0.0252 | 45/45 | 0.0033 |

### T04 Ana ölçüt ağırlıkları

Uzman grubu ve her koşulun LLM paneli için grup ağırlıkları ile LLM − uzman farkı (yüzde puan). En yüksek ağırlıklı ölçüt: Experts: Quality Performance; LLM unguided_direct: Quality Performance; LLM unguided_cot: Quality Performance; LLM guided: Quality Performance. En büyük mutlak fark: unguided_direct: Financial Performance (-8.08 pp); unguided_cot: Financial Performance (-6.54 pp); guided: Quality Performance (14.65 pp).

| Criterion | Experts | LLM unguided_direct | Diff unguided_direct (pp) | LLM unguided_cot | Diff unguided_cot (pp) | LLM guided | Diff guided (pp) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Delivery and Operational Performance | 0.2147 | 0.1744 | -4.02 | 0.1864 | -2.83 | 0.1555 | -5.91 |
| Financial Performance | 0.1604 | 0.0796 | -8.08 | 0.0950 | -6.54 | 0.0721 | -8.84 |
| Quality Performance | 0.3186 | 0.3507 | 3.22 | 0.3757 | 5.72 | 0.4651 | 14.65 |
| Volume and Logistics | 0.0529 | 0.0870 | 3.41 | 0.0800 | 2.71 | 0.0451 | -0.78 |
| Compliance and Reliability | 0.2535 | 0.3083 | 5.48 | 0.2629 | 0.94 | 0.2622 | 0.87 |

### T05 Teslimat yerel ağırlıkları

Uzman grubu ve her koşulun LLM paneli için grup ağırlıkları ile LLM − uzman farkı (yüzde puan). En yüksek ağırlıklı ölçüt: Experts: D2 Pending; LLM unguided_direct: D1 Delivered; LLM unguided_cot: D1 Delivered; LLM guided: D1 Delivered. En büyük mutlak fark: unguided_direct: D1 Delivered (23.77 pp); unguided_cot: D1 Delivered (30.56 pp); guided: D1 Delivered (36.41 pp).

| Criterion | Experts | LLM unguided_direct | Diff unguided_direct (pp) | LLM unguided_cot | Diff unguided_cot (pp) | LLM guided | Diff guided (pp) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| D1 Delivered | 0.2512 | 0.4890 | 23.77 | 0.5568 | 30.56 | 0.6153 | 36.41 |
| D2 Pending | 0.3471 | 0.1329 | -21.43 | 0.1243 | -22.28 | 0.1414 | -20.58 |
| D3 Cancelled | 0.2560 | 0.1732 | -8.28 | 0.1509 | -10.51 | 0.1005 | -15.55 |
| D4 Partially delivered | 0.1457 | 0.2050 | 5.93 | 0.1680 | 2.23 | 0.1429 | -0.28 |

### T06 Global ağırlıklar

Sekiz global ağırlık (D1–D4, F, Q, V, C) ve LLM − uzman farkı. En büyük mutlak fark: unguided_direct: F Financial (-8.08 pp); unguided_cot: F Financial (-6.54 pp); guided: Q Quality (14.65 pp).

### T07 Model düzeyinde ana ağırlıklar

Her modelin 5 tekrarının geometrik ortalamasından ana ölçüt ağırlıkları ve en yüksek ağırlıklı ölçüt. En yüksek ölçüte göre model sayısı: direct: Compliance 5, Quality 4; CoT: Quality 6, Compliance 2, Delivery 1; guided: Quality 9.

### T08 Uzman ağırlıklarına benzerlik

Model ve panel ağırlıklarının uzman ağırlıklarına Spearman ρ, Kendall τ, RMSE ve ortalama mutlak fark (pp) değerleri; ana (5) ve global (8) ağırlıklar için. Panel satırları:

| Condition | Spearman rho main | Kendall tau main | RMSE (pp) main | Mean abs diff (pp) main | Spearman rho global | Kendall tau global | RMSE (pp) global | Mean abs diff (pp) global |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| direct | 0.900 | 0.800 | 5.17 | 4.84 | 0.429 | 0.286 | 4.47 | 3.92 |
| CoT | 1.000 | 1.000 | 4.28 | 3.75 | 0.548 | 0.429 | 4.22 | 3.59 |
| guided | 1.000 | 1.000 | 8.11 | 6.21 | 0.571 | 0.500 | 6.67 | 4.93 |

### T09 Tekrarlar arası kararlılık

Model × koşul başına ana ağırlıkların tekrarlar arası standart sapması ve değişim katsayısı (CV) ile tekrarlarda görülen farklı en yüksek ölçüt sayısı. direct: ortalama CV 0.232, birden fazla farklı en yüksek ölçüt veren model 6/9; CoT: ortalama CV 0.317, birden fazla farklı en yüksek ölçüt veren model 8/9; guided: ortalama CV 0.118, birden fazla farklı en yüksek ölçüt veren model 1/9.

### T10 Panel bileşimi

Tüm modeller, geliştirici başına bir model (claude-haiku-4.5 ve gpt-5.2 çıkarılmış) ve gpt-5.2'siz panel için ana ağırlıklar ve uzmanlara benzerlik:

| Condition | Panel | Models | Spearman rho main | RMSE (pp) main | Spearman rho global | RMSE (pp) global |
| --- | --- | --- | --- | --- | --- | --- |
| direct | All models | 9 | 0.900 | 5.17 | 0.429 | 4.47 |
| direct | One per developer | 7 | 1.000 | 4.98 | 0.619 | 4.30 |
| direct | Without gpt-5.2 | 8 | 0.900 | 5.06 | 0.429 | 4.41 |
| CoT | All models | 9 | 1.000 | 4.28 | 0.548 | 4.22 |
| CoT | One per developer | 7 | 1.000 | 3.34 | 0.548 | 3.97 |
| CoT | Without gpt-5.2 | 8 | 1.000 | 3.68 | 0.548 | 3.98 |
| guided | All models | 9 | 1.000 | 8.11 | 0.571 | 6.67 |
| guided | One per developer | 7 | 1.000 | 7.47 | 0.571 | 6.26 |
| guided | Without gpt-5.2 | 8 | 1.000 | 7.72 | 0.667 | 6.41 |

### T11 Tedarikçi sıralamaları (oranlar)

Uzman ve panel global ağırlıklarıyla TOPSIS, ARAS ve EDAS puanları ve sıraları (teslimat göstergeleri oran). Birinci sıradaki tedarikçi:

| Ağırlıklar | TOPSIS 1. sıra | ARAS 1. sıra | EDAS 1. sıra |
| --- | --- | --- | --- |
| Experts | Epsilon_Group | Epsilon_Group | Alpha_Inc |
| LLM panel Unguided, zero-shot (direct) | Epsilon_Group | Epsilon_Group | Epsilon_Group |
| LLM panel Unguided, CoT | Epsilon_Group | Epsilon_Group | Epsilon_Group |
| LLM panel Guided (zero-shot) | Alpha_Inc | Epsilon_Group | Alpha_Inc |

| Ağırlıklar | Tedarikçi | TOPSIS | ARAS | EDAS |
| --- | --- | --- | --- | --- |
| Experts | Alpha_Inc | 2 | 2 | 1 |
| Experts | Beta_Supplies | 4 | 4 | 4 |
| Experts | Delta_Logistics | 5 | 5 | 5 |
| Experts | Epsilon_Group | 1 | 1 | 2 |
| Experts | Gamma_Co | 3 | 3 | 3 |
| LLM panel Unguided, zero-shot (direct) | Alpha_Inc | 2 | 2 | 2 |
| LLM panel Unguided, zero-shot (direct) | Beta_Supplies | 4 | 4 | 4 |
| LLM panel Unguided, zero-shot (direct) | Delta_Logistics | 5 | 5 | 5 |
| LLM panel Unguided, zero-shot (direct) | Epsilon_Group | 1 | 1 | 1 |
| LLM panel Unguided, zero-shot (direct) | Gamma_Co | 3 | 3 | 3 |
| LLM panel Unguided, CoT | Alpha_Inc | 2 | 2 | 2 |
| LLM panel Unguided, CoT | Beta_Supplies | 4 | 4 | 4 |
| LLM panel Unguided, CoT | Delta_Logistics | 5 | 5 | 5 |
| LLM panel Unguided, CoT | Epsilon_Group | 1 | 1 | 1 |
| LLM panel Unguided, CoT | Gamma_Co | 3 | 3 | 3 |
| LLM panel Guided (zero-shot) | Alpha_Inc | 1 | 2 | 1 |
| LLM panel Guided (zero-shot) | Beta_Supplies | 4 | 4 | 4 |
| LLM panel Guided (zero-shot) | Delta_Logistics | 5 | 5 | 5 |
| LLM panel Guided (zero-shot) | Epsilon_Group | 2 | 1 | 2 |
| LLM panel Guided (zero-shot) | Gamma_Co | 3 | 3 | 3 |

### T12 Sayılar ve oranlar

Uzman ve yönlendirmesiz – doğrudan (direct) panel ağırlıklarıyla, teslimat göstergeleri sayı ve oran olarak alındığında sıralar. 10 satır × 3 yöntemde sayı ve oran sıraları 0 hücrede farklıdır.

### T13 Eşit ağırlık karşılaştırması

Sekiz global ağırlık 1/8 iken sıralar. Birinci sıra: TOPSIS: Epsilon_Group; ARAS: Epsilon_Group; EDAS: Alpha_Inc.

### T14 Ana ölçüt ağırlığı duyarlılığı

Her ana ölçütün ağırlığı 0.05–0.70 arasında 0.05 adımla değiştirilir, diğer dört ana ölçüt kalanı eşit paylaşır; Teslimat ağırlığı D1–D4'e eşit bölünür. Tablo 71 satırdır (ilk satır eşit ağırlık başvurusu) ve her yöntem için sıralamayı ve birinci tedarikçiyi verir.

### T15 Duyarlılık özeti

T14'ün ana ölçüt × yöntem özeti. Birinci tedarikçinin hiç değişmediği durumlar: Financial/ARAS, Volume/TOPSIS, Volume/ARAS.

| Varied criterion | Method | Equal-weight top supplier | Top supplier at 0.05 | Top supplier at 0.70 | Distinct rankings | Levels with equal-weight top | Top supplier first differs at |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Delivery | TOPSIS | Epsilon_Group | Epsilon_Group | Alpha_Inc | 2 | 11 | 0.60 |
| Delivery | ARAS | Epsilon_Group | Epsilon_Group | Alpha_Inc | 2 | 13 | 0.70 |
| Delivery | EDAS | Alpha_Inc | Epsilon_Group | Alpha_Inc | 2 | 10 | 0.05 |
| Financial | TOPSIS | Epsilon_Group | Epsilon_Group | Alpha_Inc | 2 | 13 | 0.70 |
| Financial | ARAS | Epsilon_Group | Epsilon_Group | Epsilon_Group | 1 | 14 |  |
| Financial | EDAS | Alpha_Inc | Epsilon_Group | Alpha_Inc | 2 | 9 | 0.05 |
| Quality | TOPSIS | Epsilon_Group | Epsilon_Group | Alpha_Inc | 2 | 5 | 0.30 |
| Quality | ARAS | Epsilon_Group | Epsilon_Group | Alpha_Inc | 2 | 8 | 0.45 |
| Quality | EDAS | Alpha_Inc | Epsilon_Group | Alpha_Inc | 2 | 10 | 0.05 |
| Volume | TOPSIS | Epsilon_Group | Epsilon_Group | Epsilon_Group | 1 | 14 |  |
| Volume | ARAS | Epsilon_Group | Epsilon_Group | Epsilon_Group | 1 | 14 |  |
| Volume | EDAS | Alpha_Inc | Epsilon_Group | Epsilon_Group | 1 | 0 | 0.05 |
| Compliance | TOPSIS | Epsilon_Group | Alpha_Inc | Epsilon_Group | 2 | 12 | 0.05 |
| Compliance | ARAS | Epsilon_Group | Alpha_Inc | Epsilon_Group | 2 | 13 | 0.05 |
| Compliance | EDAS | Alpha_Inc | Alpha_Inc | Epsilon_Group | 2 | 3 | 0.20 |

### T16 Revizyon özeti

Model, koşul ve görev başına revizyon gereken matris sayısı ve 1–5 revizyon içinde CR ≤ 0.10'a ulaşan birikimli sayı, ortalama ilk ve son CR. Panel satırları 5.3'tedir.

### T17 Revizyon öncesi ve sonrası ağırlıklar

İlk ve son yanıtlarla panel ağırlıkları ve uzmanlara benzerlik (tablo 5.3'te).

### T18 Sıcaklık karşılaştırması (yönlendirmesiz – doğrudan (direct))

Yönlendirmesiz – doğrudan (direct) koşulda sıcaklık 0.0, 0.6 ve 1.0 için panel ana ağırlıkları, tutarlı matris sayısı ve tekrarlar arası ortalama CV. Sıcaklık kollarında 3, ana deneyde 5 tekrar vardır.

| Run tag | Temperature | Models | Delivery | Financial | Quality | Volume | Compliance | CR ≤ 0.10 main | CR ≤ 0.10 delivery | Mean CV main weights |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| temp_0.0 | 0.0 | 9 | 0.1854 | 0.0870 | 0.3538 | 0.0858 | 0.2880 | 13/27 | 14/27 | 0.148 |
| main | 0.6 | 9 | 0.1744 | 0.0796 | 0.3507 | 0.0870 | 0.3083 | 15/45 | 23/45 | 0.232 |
| temp_1.0 | 1.0 | 9 | 0.1725 | 0.0810 | 0.3297 | 0.0938 | 0.3230 | 10/27 | 14/27 | 0.230 |

### T20 Revizyon sonrası mantık korunumu

Revize edilen zincirlerde son yanıtların ilk yanıtlarla karşılaştırılması ve hakem kararları. Panel toplamı: yes 23, partly 24, no 40 (son yanıtları veren revizyon başına; panel satırları 5.3'te).

### T21 Uzman ağırlıklı sıralamayla uyum

TOPSIS, ARAS ve EDAS ile (teslimat göstergeleri oran) her koşulun LLM paneli ağırlıklarıyla ve eşit ağırlıklarla elde edilen tedarikçi sıralarının uzman ağırlıklarıyla elde edilen sıralarla Spearman ρ ve Kendall τ değerleri ile sırası farklı olan tedarikçi sayısı (sıralar T11 ve T13'teki gibidir).

| Method | Weights | Spearman rho | Kendall tau | Suppliers with different rank |
| --- | --- | --- | --- | --- |
| TOPSIS | LLM panel Unguided, zero-shot (direct) | 1.000 | 1.000 | 0 |
| TOPSIS | LLM panel Unguided, CoT | 1.000 | 1.000 | 0 |
| TOPSIS | LLM panel Guided (zero-shot) | 0.900 | 0.800 | 2 |
| TOPSIS | Equal (1/8) | 1.000 | 1.000 | 0 |
| ARAS | LLM panel Unguided, zero-shot (direct) | 1.000 | 1.000 | 0 |
| ARAS | LLM panel Unguided, CoT | 1.000 | 1.000 | 0 |
| ARAS | LLM panel Guided (zero-shot) | 1.000 | 1.000 | 0 |
| ARAS | Equal (1/8) | 1.000 | 1.000 | 0 |
| EDAS | LLM panel Unguided, zero-shot (direct) | 0.900 | 0.800 | 2 |
| EDAS | LLM panel Unguided, CoT | 0.900 | 0.800 | 2 |
| EDAS | LLM panel Guided (zero-shot) | 1.000 | 1.000 | 0 |
| EDAS | Equal (1/8) | 1.000 | 1.000 | 0 |

### T22 Yanıtlar ve yönlendirmeli istemdeki sıra

Yönlendirmeli istemde yazan sıranın yanıtlarda izlenip izlenmediği (sıra istem dosyasından okunur: ana ölçütlerde `priority_guidance` içindeki numaralı sıra; teslimatta yalnız birinci ölçüt, “PRIMARY positive indicator” olarak anılan teslim edilen siparişler). Panel (guided): main: 45 yanıtın 40 tanesinde sıralama istemdeki sırayla aynı, 42 tanesinde istemdeki birinci ölçüt birinci; delivery: 45 yanıtın – tanesinde sıralama istemdeki sırayla aynı, 45 tanesinde istemdeki birinci ölçüt birinci. Doğrudan ve CoT yanıtları ile uzmanların bireysel matrisleri bu sırayı görmemiştir ve yalnız karşılaştırma (reference) içindir.

| Respondents | Guidance | Technique | Condition | Comparison | Task | Replies | Ranking equals prompted order | Prompted first criterion ranked first | Mean Spearman rho with prompted order |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| claude-haiku-4.5 | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| claude-sonnet-4.6 | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| deepseek-v3.2 | guided | zero-shot | guided | prompted | main | 5 | 2 | 2 | 0.920 |
| gemini-3.1-flash-lite | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| glm-5 | guided | zero-shot | guided | prompted | main | 5 | 4 | 5 | 0.980 |
| gpt-4o | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| gpt-5.2 | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| minimax-m2.5 | guided | zero-shot | guided | prompted | main | 5 | 4 | 5 | 0.980 |
| qwen3.5-397b | guided | zero-shot | guided | prompted | main | 5 | 5 | 5 | 1.000 |
| Panel | guided | zero-shot | guided | prompted | main | 45 | 40 | 42 | 0.987 |
| Panel | unguided | zero-shot | direct | reference (order not shown) | main | 45 | 6 | 21 | 0.776 |
| Panel | unguided | CoT | CoT | reference (order not shown) | main | 45 | 7 | 24 | 0.711 |
| Experts (individual matrices) |  |  |  | reference (order not shown) | main | 7 | 1 | 2 | 0.671 |
| claude-haiku-4.5 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| claude-sonnet-4.6 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| deepseek-v3.2 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| gemini-3.1-flash-lite | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| glm-5 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| gpt-4o | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| gpt-5.2 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| minimax-m2.5 | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| qwen3.5-397b | guided | zero-shot | guided | prompted | delivery | 5 |  | 5 |  |
| Panel | guided | zero-shot | guided | prompted | delivery | 45 |  | 45 |  |
| Panel | unguided | zero-shot | direct | reference (order not shown) | delivery | 45 |  | 27 |  |
| Panel | unguided | CoT | CoT | reference (order not shown) | delivery | 45 |  | 41 |  |
| Experts (individual matrices) |  |  |  | reference (order not shown) | delivery | 7 |  | 3 |  |

### Şekiller

- F1 (`figures/F1_global_weight_diff_heatmap.png`): model − uzman global ağırlık farkı (pp), koşul başına bir panel.
- F2 (`figures/F2_cr_main_dotplot.png`): ana ölçüt matrislerinin CR değerleri, model ve koşul başına; kesikli çizgi CR = 0.10.

## 7. Ekler listesi

`results/report/appendix/` klasöründeki dosyalar (satır sayısı başlık hariç):

| Dosya | Satır | Açıklama |
| --- | --- | --- |
| appendix/A04_group_matrices.csv | 36 | A04 Uzman ve koşul başına LLM paneli geometrik ortalama matrisleri; ağırlık, λmax, CI, CR. |
| appendix/A05_model_matrices.csv | 243 | A05 Model × koşul × görev geometrik ortalama matrisleri (5 tekrar); ağırlık, λmax, CI, CR. |
| appendix/A06_group_matrices_after_revision.csv | 18 | A06 Revizyon sonrası (direct, CoT) LLM paneli geometrik ortalama matrisleri. |
| appendix/A07_run_level.csv | 270 | A07 Çağrı düzeyinde tablo: CR, ağırlıklar, sağlayıcı, token, maliyet, ayrıştırma bayrakları, revizyon gereksinimi, revizyon sayısı, son CR, hakem kararları. |
| appendix/A08_judge_feedback.csv | 250 | A08 Revision_final hakem geri bildirimleri ve kararları (her revizyon çağrısı). |
| appendix/cot_reasoning.csv | 90 | A03 CoT koşulunda modellerin verdiği sıralama ve gerekçe metni. |
| appendix/matrices_expert.csv | 287 | A02 Her uzman matrisinin tüm hücreleri ve CR'si. |
| appendix/matrices_llm.csv | 5535 | A01 Seçilen her LLM matrisinin tüm hücreleri (uzun biçim) ve matris CR'si. |
| appendix/README.md |  | Bu klasördeki ve tables/ klasöründeki dosyaların İngilizce listesi (açıklama ve satır sayısı). |

Denetim çıktıları `results/report/audit/` altındadır (`audit.md` ve a–k bölümlerinin CSV dosyaları).

## 8. Sınırlılıklar

- **gpt-5.2 sıcaklığı.** Model sıcaklığı uygulamıyor: T = 0 ile 5 çağrıda 5 farklı yanıt kümesi. Bu modelin sıcaklık kollarındaki ve ana deneydeki değişkenliği istenen sıcaklığa bağlanamaz.
- **minimax-m2.5 reasoning.** Reasoning kapatılamadı; ana deneyde ortalama 691.6 reasoning token üretildi. Diğer modellerde reasoning kapalıydı (gpt-4o için parametre gönderilmedi).
- **Sağlayıcı yönlendirmesi sabitlenmedi.** OpenRouter aynı modeli farklı sağlayıcılara yönlendirdi; ana deneyde 3 model birden fazla sağlayıcıdan yanıt aldı (3.2-e). Sağlayıcılar arası farklar ayrıca incelenmedi.
- **Tekrar sayısı.** Her model × koşul × görev hücresinde 5, sıcaklık kollarında 3 tekrar vardır. Model düzeyindeki ortalama ve dağılım ölçüleri bu az sayıda gözleme dayanır.
- **Dil.** Yönlendirmesiz – doğrudan (direct) ve CoT istemleri İngilizcedir; uzmanlara sorular Türkçe sorulmuştur. Guided isteminde talimatlar İngilizce; ana ölçüt adları, açıklamaları ve KPI raporu Türkçedir.
- **Olumsuz alt ölçütlerde “önem” belirsizliği.** Sorular “hangisi daha önemli” diye sorar. Performans matrisinde D2, D3, D4, Q, C maliyet yönlüdür (D2–D4 bekleyen, iptal edilen ve kısmen teslim edilen siparişlerdir). Soru metni, önemin istenen ya da istenmeyen bir durumun izlenmesi anlamında mı sorulduğunu belirtmez.
- **Revizyon kapsamı.** Revizyon yalnız yönlendirmesiz – doğrudan (direct) ve CoT koşullarında yapıldı. Guided koşulunda CR > 0.10 olan matris sayısı: ana görevde 2, teslimat görevinde 0. Bu matrisler revize edilmedi.
- **Hakem bir LLM'dir.** Mantık korunumu kararları tek bir modelden gelir; dosyalarda bu kararların insan değerlendirmesiyle karşılaştırılması yoktur.
- **Pilotlar dışarıda.** `pilot` (10 çağrı) ve revizyon pilotları sonuçlara alınmadı: `revision_judge` 245 panel + 458 hakem çağrısı; `revision_v2` 6 panel + 12 hakem çağrısı; `rev_final` 7 panel + 14 hakem çağrısı; `rev_test_none` 2 panel + 4 hakem çağrısı. Hakem modeli: `revision_judge` x-ai/grok-4.20; `revision_v2` x-ai/grok-4.7; `rev_final` x-ai/grok-4.7; `rev_test_none` x-ai/grok-4.7. `revision_judge` ilk protokoldür; diğerleri kısa denemelerdir.

## 9. Harcama özeti

Maliyetler OpenRouter yanıtındaki `usage.cost` alanından toplanmıştır (USD). Yedek dosyalar ana kayıtların kopyası olduğu için toplama alınmadı.

| Aşama | Maliyet (USD) |
| --- | --- |
| Parametre sınamaları | 0.0241 |
| Ön deneme (pilot) | 0.0142 |
| Ana deney (main) | 1.4783 |
| Sıcaklık kolu temp_0.0 | 0.0845 |
| Sıcaklık kolu temp_1.0 | 0.0948 |
| Revizyon pilotu revision_judge (panel + hakem) | 1.0404 |
| Revizyon pilotu revision_v2 (panel + hakem) | 0.2015 |
| Revizyon pilotu rev_final (panel + hakem) | 0.1557 |
| Revizyon pilotu rev_test_none (panel + hakem) | 0.0356 |
| Revizyon revision_final, panel | 0.6633 |
| Revizyon revision_final, hakem | 0.4735 |
| **Toplam** | 4.2660 |
