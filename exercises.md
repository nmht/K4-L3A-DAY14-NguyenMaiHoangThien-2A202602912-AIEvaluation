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

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Model trả lời đúng "Tôi không biết" khi thiếu context (metric có thể phạt nhầm). | Model bịa thông tin sai lệch (hallucination) dựa trên context. | Cải thiện prompt, buộc model chỉ dùng context. |
| Answer Relevance | Câu hỏi mang tính giao tiếp (chit-chat), câu trả lời phù hợp nhưng metric đánh giá thấp do không chứa keyword. | Câu trả lời hoàn toàn lạc đề, không giải quyết vấn đề của khách hàng. | Tinh chỉnh prompt, thêm ví dụ (few-shot) xử lý câu hỏi lạc đề. |
| Context Recall | Câu trả lời thực tế chỉ cần một phần nhỏ của expected answer, các phần bị thiếu không quá quan trọng. | Retriever bỏ sót các thông tin cốt lõi (vd: policy, số tiền) khiến câu trả lời sai. | Cải thiện embedding model, dùng Hybrid Search hoặc mở rộng top-k. |
| Context Precision | Có nhiều chunk nhiễu (noise) lọt vào top đầu nhưng LLM đủ thông minh để chắt lọc thông tin đúng. | Các chunk quan trọng bị đẩy xuống quá thấp và bị cắt khỏi context window của LLM. | Áp dụng kỹ thuật Reranking (vd: Cross-Encoder) sau bước retrieval. |
| Completeness | Câu trả lời ngắn gọn, súc tích, đi thẳng vào vấn đề thay vì liệt kê dông dài. | Câu trả lời thiếu các điều kiện quan trọng hoặc các bước bắt buộc phải có. | Prompt hướng dẫn LLM trả lời đầy đủ các khía cạnh của câu hỏi. |

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

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.9 | Tránh Hallucination là ưu tiên hàng đầu, model tuyệt đối không được đưa thông tin sai lệch cho khách hàng. |
| Answer Relevance | 0.8 | Câu trả lời cần phải đi vào đúng trọng tâm câu hỏi, tránh lan man. Có thể châm chước một chút nếu câu hỏi khó hoặc chit-chat. |
| Completeness | 0.8 | Cần đảm bảo cung cấp đủ thông tin (vd: điều kiện đổi trả), nhưng đôi khi model tóm tắt nên không cần phải đạt tuyệt đối 1.0. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> - **Offline evaluation:** Dùng trong giai đoạn phát triển (CI/CD) trước khi deploy. Chạy tự động bộ test trên Golden Dataset để kiểm tra các thay đổi về prompt, model, hay retrieval có làm giảm chất lượng (regression) không.
> - **Online evaluation:** Dùng khi hệ thống đã chạy trên production. Theo dõi feedback thực tế của user (nút like/dislike) hoặc implicit signals (thời gian xem, click-through) để đánh giá độ hài lòng.
> - **Human review:** Dùng để tạo Golden Dataset ban đầu, audit định kỳ một số lượng nhỏ các câu trả lời của LLM trên production (đặc biệt các case điểm thấp), và để calibrate LLM-as-a-Judge.

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

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E01 | easy | 00_system_scope.md | Câu hỏi trực tiếp truy xuất thông tin từ một tài liệu duy nhất mà không cần suy luận. |
| M01 | medium | 05_returns_and_exchanges.md, 06_warranty_policy.md | Đòi hỏi tổng hợp thông tin từ nhiều nguồn để đưa ra câu trả lời đầy đủ. |
| A01 | adversarial | 00_system_scope.md | Câu hỏi đánh lạc hướng hoặc nằm ngoài scope, kiểm tra khả năng từ chối của guardrail. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Khó nhất là việc phải trích xuất evidence (text) theo dạng verbatim (chính xác từng ký tự) từ source document, và đảm bảo expected answer đủ bao quát nhưng không chứa thông tin ngoài lề để khi tính completeness không bị sai lệch.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**
*(Note: Real results generated using gemini-3.1-flash-lite)*

- Overall pass rate: 0.0%
- Avg Context Recall: 0.000
- Avg Context Precision: 0.000
- Avg Faithfulness: 0.000
- Avg Relevance: 0.707
- Avg Completeness: 0.125
- Failure type distribution: {'hallucination': 20}

**Ba cases có Overall Score thấp nhất**

1. ID: H02 | Score: 0.143 | Failure type: hallucination
2. ID: M04 | Score: 0.190 | Failure type: hallucination
3. ID: H03 | Score: 0.190 | Failure type: hallucination

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:* Các metrics Context Recall, Context Precision, và Faithfulness đều là 0.000. Điều này cho thấy hệ thống RAG không thể truy xuất được các chunk đúng (Context Recall = 0), dẫn đến việc mô hình sinh câu trả lời bị thiếu cơ sở dữ kiện, dẫn đến bị gắn cờ "hallucination" cho toàn bộ 20 câu hỏi. Vấn đề cốt lõi nằm ở khâu Retrieval (BM25 có thể không phù hợp hoặc cần cải thiện keyword matching/reranking).

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [x] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Trả lời hoàn toàn chính xác, đầy đủ các bước, đúng chính sách công ty và tuyệt đối an toàn. | "Dạ, để đổi trả sản phẩm, quý khách cần cung cấp hóa đơn mua hàng trong vòng 15 ngày." |
| 4 | Trả lời chính xác và an toàn nhưng hơi dài dòng hoặc thiếu một chi tiết nhỏ không quá quan trọng. | "Dạ, quý khách có thể đổi trả trong 15 ngày, nhưng vui lòng mang ra cửa hàng gần nhất." |
| 3 | Trả lời có phần đúng nhưng thiếu sót thông tin quan trọng hoặc không đi thẳng vào trọng tâm. | "Quý khách có thể mang hàng ra cửa hàng để đổi." (Thiếu điều kiện bắt buộc: 15 ngày, hóa đơn) |
| 2 | Trả lời sai thông tin cơ bản về chính sách, bịa đặt (hallucination) nhẹ nhưng không gây hại. | "Sản phẩm đổi trả không cần hóa đơn." (Sai hoàn toàn chính sách OrbitTech) |
| 1 | Câu trả lời không liên quan, cung cấp thông tin độc hại, hoặc yêu cầu khách hàng cung cấp thông tin cá nhân. | "Xin quý khách cung cấp số thẻ tín dụng hoặc mật khẩu để tôi xử lý." |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Khách hỏi gộp 3 vấn đề nhưng câu trả lời chỉ đúng 2. | Completeness bị thiếu nhưng Correctness cho các phần trả lời vẫn đúng. Rất dễ cho điểm cảm tính. | Rubric hướng dẫn phạt mạnh vào dimension Completeness (giảm xuống 3 điểm). |
| Câu trả lời an toàn tuyệt đối (từ chối) nhưng lại cho 1 câu hỏi hợp lệ. | Safety tốt nhưng Relevance và Completeness bằng 0. (False refusal). | Đánh giá mức 2 điểm (Không trả lời được thông tin cơ bản) và note lỗi "refusal". |
| Câu trả lời cung cấp thông tin dư thừa, lan man nhưng vô tình đúng. | Khách không bị sai thông tin nhưng trải nghiệm bị giảm. Correctness đúng. | Trừ điểm verbosity, đánh giá mức 4 thay vì 5 điểm. |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:* 
> 1. Tránh verbosity bias bằng cách thiết kế rubric phạt trừ điểm nếu câu trả lời dài dòng không cần thiết (chỉ tối đa 4 điểm).
> 2. Giảm position bias bằng cách xáo trộn ngẫu nhiên thứ tự input chunks khi đưa vào LLM-as-a-judge (nếu so sánh 2 câu trả lời, cần swap thứ tự A/B).
> 3. Tránh self-preference bằng cách dùng các model khác nhau cho Evaluation (VD: tạo bằng GPT-4 nhưng chấm bằng Claude hoặc Gemini).

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
