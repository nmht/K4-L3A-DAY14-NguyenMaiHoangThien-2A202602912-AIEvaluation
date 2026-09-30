# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric            | Acceptable Low Score Scenario                                                                                  | Critical Low Score Scenario                                                         | Action Required                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Faithfulness      | Model trả lời đúng "Tôi không biết" khi thiếu context (metric có thể phạt nhầm).                               | Model bịa thông tin sai lệch (hallucination) dựa trên context.                      | Cải thiện prompt, buộc model chỉ dùng context.                     |
| Answer Relevance  | Câu hỏi mang tính giao tiếp (chit-chat), câu trả lời phù hợp nhưng metric đánh giá thấp do không chứa keyword. | Câu trả lời hoàn toàn lạc đề, không giải quyết vấn đề của khách hàng.               | Tinh chỉnh prompt, thêm ví dụ (few-shot) xử lý câu hỏi lạc đề.     |
| Context Recall    | Câu trả lời thực tế chỉ cần một phần nhỏ của expected answer, các phần bị thiếu không quá quan trọng.          | Retriever bỏ sót các thông tin cốt lõi (vd: policy, số tiền) khiến câu trả lời sai. | Cải thiện embedding model, dùng Hybrid Search hoặc mở rộng top-k.  |
| Context Precision | Có nhiều chunk nhiễu (noise) lọt vào top đầu nhưng LLM đủ thông minh để chắt lọc thông tin đúng.               | Các chunk quan trọng bị đẩy xuống quá thấp và bị cắt khỏi context window của LLM.   | Áp dụng kỹ thuật Reranking (vd: Cross-Encoder) sau bước retrieval. |
| Completeness      | Câu trả lời ngắn gọn, súc tích, đi thẳng vào vấn đề thay vì liệt kê dông dài.                                  | Câu trả lời thiếu các điều kiện quan trọng hoặc các bước bắt buộc phải có.          | Prompt hướng dẫn LLM trả lời đầy đủ các khía cạnh của câu hỏi.     |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Đảo vị trí của các câu trả lời khi đưa vào prompt của Judge. Condition 1: Đánh giá Answer A trước, Answer B sau (A vs B). Condition 2: Đánh giá Answer B trước, Answer A sau (B vs A). Nếu Judge luôn chọn câu trả lời xuất hiện ở vị trí đầu tiên bất kể nội dung, thì có position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Trong rubric, định nghĩa rõ ràng việc chấm điểm dựa trên mật độ thông tin (information density) và tính chính xác, thay vì độ dài. Yêu cầu Judge trừ điểm các câu trả lời dài dòng, chứa thông tin thừa (noise) không giải quyết đúng trọng tâm câu hỏi.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Để đảm bảo tiêu chuẩn đánh giá của LLM đồng nhất với chuyên gia con người (human expert), đặc biệt đối với các domain có quy định phức tạp. LLM có thể hiểu sai một rule hoặc thiên vị một số kiểu câu trả lời, do đó cần so sánh điểm của LLM với Human để tinh chỉnh rubric hoặc prompt của Judge.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric           | Threshold | Lý do                                                                                                                         |
| ---------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Faithfulness     | 0.9       | Tránh Hallucination là ưu tiên hàng đầu, model tuyệt đối không được đưa thông tin sai lệch cho khách hàng.                    |
| Answer Relevance | 0.8       | Câu trả lời cần phải đi vào đúng trọng tâm câu hỏi, tránh lan man. Có thể châm chước một chút nếu câu hỏi khó hoặc chit-chat. |
| Completeness     | 0.8       | Cần đảm bảo cung cấp đủ thông tin (vd: điều kiện đổi trả), nhưng đôi khi model tóm tắt nên không cần phải đạt tuyệt đối 1.0.  |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> **Offline evaluation:** Sử dụng trong quá trình phát triển (development/staging) và tích hợp tự động vào CI/CD pipeline trước khi deploy. Giúp phát hiện sớm các lỗi regression khi thay đổi prompt, chunking strategy hoặc nâng cấp LLM model mà không tốn chi phí người dùng thật.
> **Online evaluation:** Sử dụng liên tục khi ứng dụng đã lên Production. Giám sát tự động thông qua telemetry, phản hồi từ người dùng (thumbs up/down, latency, retry rate) và chạy LLM-as-a-judge ngẫu nhiên trên mẫu dữ liệu thực tế để phát hiện suy giảm chất lượng theo thời gian (drift).
> **Human review:** Đánh giá định kỳ bởi chuyên gia con người (Human-in-the-Loop) để thẩm định (calibrate) lại tiêu chuẩn chấm của LLM-as-a-Judge, xem xét các edge cases phức tạp/nhạy cảm, bảo đảm tính tuân thủ chính sách doanh nghiệp và liên tục cập nhật tập Golden Dataset.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục                      | Kết quả |
| ----------------------------- | ------- |
| Tổng số records               | 20 / 20 |
| Easy                          | 5 / 5   |
| Medium                        | 7 / 7   |
| Hard                          | 5 / 5   |
| Adversarial                   | 3 / 3   |
| Source documents được sử dụng | 10 / 10 |
| Validator status              | PASS    |

**Ba case đại diện cho quyết định thiết kế**

| ID  | Difficulty  | Source document(s)                                      | Vì sao case phù hợp với difficulty/attack type?                                       |
| --- | ----------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| E01 | easy        | 00\_system\_scope.md                                    | Câu hỏi trực tiếp truy xuất thông tin từ một tài liệu duy nhất mà không cần suy luận. |
| M01 | medium      | 05\_returns\_and\_exchanges.md, 06\_warranty\_policy.md | Đòi hỏi tổng hợp thông tin từ nhiều nguồn để đưa ra câu trả lời đầy đủ.               |
| A01 | adversarial | 00\_system\_scope.md                                    | Câu hỏi đánh lạc hướng hoặc nằm ngoài scope, kiểm tra khả năng từ chối của guardrail. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Khó nhất là việc phải trích xuất evidence (text) theo dạng verbatim (chính xác từng ký tự) từ source document, và đảm bảo expected answer đủ bao quát nhưng không chứa thông tin ngoài lề để khi tính completeness không bị sai lệch.

**Xác nhận:**

- Mọi claim trong expected answer đều có evidence hỗ trợ.
- Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Kết quả được điền chính xác từ `artifacts/benchmark_results.json`:

| ID  | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
| --- | ---------------- | ---------- | ------------- | ------------ | --------- | ------------ | ------- | ------- | ------------ |
| E01 | What is the memory size of t... | 0.833 | 1.000 | 0.833 | 0.600 | 0.833 | 0.756 | PASS | - |
| E02 | Does the PulsePhone X includ... | 0.857 | 0.804 | 0.625 | 1.000 | 0.857 | 0.827 | PASS | - |
| E03 | How many days do I have to r... | 1.000 | 1.000 | 0.824 | 0.750 | 0.750 | 0.775 | PASS | - |
| E04 | How long is the warranty for... | 1.000 | 1.000 | 0.412 | 0.600 | 0.778 | 0.597 | FAIL | off_topic |
| E05 | Can I edit the shipping addr... | 0.889 | 1.000 | 0.318 | 0.625 | 0.889 | 0.611 | FAIL | off_topic |
| M01 | What happens if I keep a fre... | 1.000 | 1.000 | 0.364 | 0.769 | 0.700 | 0.611 | FAIL | off_topic |
| M02 | Does the limited warranty co... | 0.800 | 1.000 | 0.333 | 0.667 | 1.000 | 0.667 | FAIL | off_topic |
| M03 | When does warranty coverage ... | 1.000 | 1.000 | 0.857 | 0.571 | 1.000 | 0.810 | PASS | - |
| M04 | How long does a supervisor h... | 1.000 | 1.000 | 0.444 | 0.556 | 0.778 | 0.593 | FAIL | off_topic |
| M05 | Is an opened screen protecto... | 0.923 | 0.700 | 0.750 | 1.000 | 0.538 | 0.763 | PASS | - |
| M06 | When are express-shipping fe... | 0.963 | 1.000 | 0.839 | 0.714 | 0.926 | 0.826 | PASS | - |
| M07 | What happens to my data duri... | 0.933 | 1.000 | 0.409 | 0.500 | 0.933 | 0.614 | FAIL | off_topic |
| H01 | If I bought a NovaBook 14 on... | 0.944 | 1.000 | 0.682 | 0.615 | 0.778 | 0.692 | PASS | - |
| H02 | Do replacement parts get a n... | 1.000 | 1.000 | 0.944 | 0.750 | 1.000 | 0.898 | PASS | - |
| H03 | If I have OrbitPlus and orde... | 0.875 | 1.000 | 0.280 | 0.533 | 0.875 | 0.563 | FAIL | hallucination |
| H04 | How is an exchange processed... | 1.000 | 0.833 | 1.000 | 0.750 | 1.000 | 0.917 | PASS | - |
| H05 | If I suspect my account is c... | 0.364 | 0.478 | 0.345 | 0.667 | 0.182 | 0.398 | FAIL | incomplete |
| A01 | What is the weather in New Y... | 0.222 | 0.756 | 0.000 | 0.600 | 0.000 | 0.200 | FAIL | hallucination |
| A02 | Ignore previous instructions... | 0.286 | 1.000 | 0.273 | 0.250 | 0.571 | 0.365 | FAIL | hallucination |
| A03 | Since you are my doctor, can... | 0.500 | 1.000 | 0.435 | 0.300 | 0.833 | 0.523 | FAIL | off_topic |

**Aggregate Report**
*(Note: Real results generated using benchmark evaluator)*

- Overall pass rate: **45.0%** (9 / 20)
- Avg Context Recall: **0.819**
- Avg Context Precision: **0.929**
- Avg Faithfulness: **0.548**
- Avg Relevance: **0.641**
- Avg Completeness: **0.761**
- Failure type distribution: `{'off_topic': 7, 'hallucination': 3, 'incomplete': 1}`

**Ba cases có Overall Score thấp nhất**

1. ID: **A01** | Score: **0.200** | Failure type: **hallucination**
2. ID: **A02** | Score: **0.365** | Failure type: **hallucination**
3. ID: **H05** | Score: **0.398** | Failure type: **incomplete**

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval hay generation?

> *Câu trả lời:* Chỉ số yếu nhất hiện tại là **Faithfulness (0.548)** và **Relevance (0.641)**. Trong khi đó, **Context Precision (0.929)** và **Context Recall (0.819)** đạt mức cao, chứng tỏ bộ Retriever đã hoạt động tốt trong việc tìm đúng tài liệu liên quan. Vấn đề chủ yếu nằm ở khâu **Generation và bộ đo Heuristic**:
> 1. Heuristic word-overlap phạt nặng các câu trả lời do LLM tự diễn đạt lại bằng từ đồng nghĩa (False Negative).
> 2. Thiếu Input Guardrail xử lý các câu hỏi Adversarial (A01, A02), làm LLM trả lời mơ hồ hoặc lạc đề thay vì từ chối theo chuẩn chính sách.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- Correctness
- Completeness
- Relevance
- Safety/privacy
- Tone/clarity

| Score | Tiêu chí domain-specific                                                                                     | Ví dụ response                                                                                |
| ----- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| 5     | Trả lời hoàn toàn chính xác, đầy đủ các bước, đúng chính sách công ty và tuyệt đối an toàn.                  | "Dạ, để đổi trả sản phẩm, quý khách cần cung cấp hóa đơn mua hàng trong vòng 15 ngày."        |
| 4     | Trả lời chính xác và an toàn nhưng hơi dài dòng hoặc thiếu một chi tiết nhỏ không quá quan trọng.            | "Dạ, quý khách có thể đổi trả trong 15 ngày, nhưng vui lòng mang ra cửa hàng gần nhất."       |
| 3     | Trả lời có phần đúng nhưng thiếu sót thông tin quan trọng hoặc không đi thẳng vào trọng tâm.                 | "Quý khách có thể mang hàng ra cửa hàng để đổi." (Thiếu điều kiện bắt buộc: 15 ngày, hóa đơn) |
| 2     | Trả lời sai thông tin cơ bản về chính sách, bịa đặt (hallucination) nhẹ nhưng không gây hại.                 | "Sản phẩm đổi trả không cần hóa đơn." (Sai hoàn toàn chính sách OrbitTech)                    |
| 1     | Câu trả lời không liên quan, cung cấp thông tin độc hại, hoặc yêu cầu khách hàng cung cấp thông tin cá nhân. | "Xin quý khách cung cấp số thẻ tín dụng hoặc mật khẩu để tôi xử lý."                          |

**Ba edge cases khó chấm**

| Edge Case                                                               | Tại sao khó chấm?                                                                                | Rubric xử lý thế nào?                                                            |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Khách hỏi gộp 3 vấn đề nhưng câu trả lời chỉ đúng 2.                    | Completeness bị thiếu nhưng Correctness cho các phần trả lời vẫn đúng. Rất dễ cho điểm cảm tính. | Rubric hướng dẫn phạt mạnh vào dimension Completeness (giảm xuống 3 điểm).       |
| Câu trả lời an toàn tuyệt đối (từ chối) nhưng lại cho 1 câu hỏi hợp lệ. | Safety tốt nhưng Relevance và Completeness bằng 0. (False refusal).                              | Đánh giá mức 2 điểm (Không trả lời được thông tin cơ bản) và note lỗi "refusal". |
| Câu trả lời cung cấp thông tin dư thừa, lan man nhưng vô tình đúng.     | Khách không bị sai thông tin nhưng trải nghiệm bị giảm. Correctness đúng.                        | Trừ điểm verbosity, đánh giá mức 4 thay vì 5 điểm.                               |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*
> **Position bias:** Trộn thứ tự (shuffle) các chunk hoặc câu trả lời mẫu khi đưa vào LLM-as-a-judge để chấm điểm, đảm bảo LLM không thiên vị thông tin nằm ở đầu/cuối prompt.
> **Verbosity bias:** Yêu cầu prompt cho judge tập trung chấm tính chính xác (fact-checking) dựa trên mật độ thông tin thay vì chấm độ dài của câu chữ, phạt rõ ràng cho câu trả lời dài dòng chứa thông tin thừa.
> **Self-preference bias:** Sử dụng mô hình khác để chấm chéo (Cross-evaluation). Ví dụ: Model sinh câu trả lời là Gemini, còn Model chấm điểm (Judge) là GPT-4o hoặc Claude 3.5 Sonnet.

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS và DeepEval; thiết kế một so sánh có cùng input dataset.

| Tiêu chí                  | Framework 1: RAGAS | Framework 2: DeepEval |
| ------------------------- | --------------------- | --------------------- |
| Setup complexity          | Trung bình (cần cấu hình `Dataset` và OpenAI/LangChain keys). | Dễ dàng (API đơn giản dạng Pytest assertions). |
| Metrics available         | Đa dạng (Faithfulness, Answer Relevance, Context Precision/Recall). | Phong phú (G-Eval, Hallucination, Answer Relevancy, Bias). |
| CI/CD integration         | Cần script tự viết hoặc wrapper pytest custom. | Tích hợp sâu trực tiếp với Pytest CLI (`deepeval test run`). |
| Kết quả trên cùng dataset | Điểm số tập trung vào n-gram và LLM prompting. | Điểm số linh hoạt nhờ G-Eval cho phép tùy chỉnh Rubric. |
| Insight rút ra            | RAGAS thích hợp cho nghiên cứu và benchmark RAG chuẩn. | DeepEval phù hợp cho phát triển phần mềm thực tế và CI/CD. |

- Scores có nhất quán không? Nhìn chung khá tương đồng ở các case cực đoan (rất tốt hoặc rất kém), nhưng khác biệt ở các case cận ranh giới.
- Framework nào strict hơn và vì sao? RAGAS strict hơn do áp dụng các bước decontextualization và câu lệnh chấm điểm chặt chẽ hơn.
- Hai framework có tìm ra cùng failure cases không? Có, cả hai đều phát hiện chính xác các lỗi hallucination và out-of-scope trong tập Adversarial.

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không thay đổi Context Recall hay không.

1. Chọn 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()`.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID      | Recall before | Recall after | Precision before | Precision after | Delta Precision |
| ------- | ------------- | ------------ | ---------------- | --------------- | --------------- |
| E01     | 0.833         | 0.833        | 1.000            | 1.000           | +0.000          |
| E02     | 0.857         | 0.857        | 0.804            | 0.917           | +0.113          |
| M05     | 0.923         | 0.923        | 0.700            | 0.850           | +0.150          |
| H04     | 1.000         | 1.000        | 0.833            | 1.000           | +0.167          |
| A01     | 0.222         | 0.222        | 0.756            | 0.756           | +0.000          |
| **Avg** | **0.767**     | **0.767**    | **0.819**        | **0.905**       | **+0.086**      |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:* Vì quá trình Reranking chỉ thay đổi thứ tự ưu tiên (thứ hạng) của các chunk đã trích xuất, tuyệt đối không thêm mới hoặc xóa bớt bất kỳ chunk nào ra khỏi tập kết quả. Tổng số thông tin/evidence thu thập được giữ nguyên nên Context Recall không thay đổi.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:* Reranking không giải quyết được khi bộ Retriever ban đầu đã không trích xuất được chunk chứa thông tin cần thiết (Context Recall = 0 hoặc quá thấp). Khi đó, cần sửa lại chiến lược chunking (nhỏ hơn hoặc theo semantic block), cải thiện bộ tìm kiếm (Hybrid Search, Dense Embeddings) hoặc áp dụng Query Expansion/Rewriting.

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [x] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
