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

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Có thể chấp nhận dưới 0.8 với câu hỏi ngoài phạm vi hoặc câu trả lời từ chối an toàn, không đưa ra claim quan trọng. | Dưới 0.6 khi claim về giá, chính sách, bảo hành hoặc đơn hàng không được retrieved context hỗ trợ; đây là rủi ro hallucination. | Kiểm tra từng claim với evidence; cải thiện prompt grounding và thêm test chống hallucination. |
| Answer Relevance | Có thể chấp nhận khi câu hỏi mơ hồ và hệ thống cần yêu cầu người dùng làm rõ. | Dưới 0.6 khi câu trả lời lạc chủ đề, nhầm intent hoặc không giải quyết vấn đề khách hàng. | Kiểm tra intent classification và prompt; bổ sung case đa intent, mơ hồ và off-topic. |
| Context Recall | Có thể chấp nhận với câu hỏi đơn giản chỉ cần một phần evidence hoặc khi hệ thống cần hỏi thêm thông tin. | Dưới 0.6 khi context thiếu policy, điều kiện hoặc ngoại lệ cần thiết, dẫn đến câu trả lời thiếu hoặc sai. | Kiểm tra corpus/chunking; cải thiện query rewrite, top-k và synonym của thuật ngữ domain. |
| Context Precision | Có thể chấp nhận nếu top-k có một vài chunk dư thừa nhưng evidence đúng vẫn rõ và không gây nhiễu. | Dưới 0.6 khi phần lớn chunks không liên quan hoặc evidence sai đứng trước evidence đúng. | Rerank hoặc giảm top-k, cải thiện BM25/query expansion và theo dõi theo từng loại câu hỏi. |
| Completeness | Có thể chấp nhận nếu chỉ thiếu chi tiết tùy chọn nhưng câu trả lời vẫn đủ phần cốt lõi và hướng dẫn an toàn. | Dưới 0.6 khi bỏ sót điều kiện, deadline, giới hạn, bước xử lý hoặc escalation cần thiết. | So sánh answer với từng claim trong expected answer; thêm checklist bắt buộc và test hard/adversarial. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Dùng cùng một tập câu hỏi và cùng hai câu trả lời A/B có chất lượng tương đương. Ở condition 1, đặt A trước B; ở condition 2, đảo thành B trước A. Giữ nguyên prompt, rubric và model judge, chạy nhiều lần rồi so sánh tỷ lệ answer đứng thứ nhất được chọn. Nếu cùng một answer thường thắng khi đổi vị trí, đó là position bias. Nên randomize thứ tự cho mỗi trial và báo cáo win-rate cùng khoảng tin cậy.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Chấm theo các tiêu chí riêng như correctness, evidence/faithfulness, completeness và clarity; không dùng độ dài làm proxy cho chất lượng. Quy định rõ câu trả lời ngắn nhưng đủ ý phải đạt điểm cao, còn nội dung dài nhưng lặp lại hoặc không liên quan phải bị trừ điểm. Có thể giới hạn answer vào cùng format và yêu cầu judge bỏ qua số từ khi chấm.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Human labels cung cấp chuẩn tham chiếu để đo agreement, phát hiện judge chấm quá dễ/quá nghiêm và kiểm tra các nhóm lỗi mà judge thường bỏ sót. Calibration cũng giúp chọn threshold, điều chỉnh rubric và theo dõi drift khi model hoặc domain thay đổi. Nếu agreement giảm, cần review lại các case disagreement thay vì tin tuyệt đối vào điểm tự động.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.80 | Ngăn hallucination trong các câu trả lời chính sách, bảo hành và đơn hàng; đây là metric an toàn quan trọng. |
| Answer Relevance | 0.75 | Đảm bảo assistant trả lời đúng intent và không làm khách hàng mất thời gian với nội dung lạc đề. |
| Completeness | 0.75 | Bảo đảm không bỏ sót điều kiện, giới hạn hoặc bước escalation cần thiết trong support answer. |

Ngoài threshold trung bình, deployment phải bị block nếu có regression lớn hơn 0.05 so với baseline hoặc có case critical dưới 0.6.

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Offline evaluation chạy trước khi merge hoặc deploy, dùng golden dataset cố định để kiểm tra regression nhanh, reproducible và không ảnh hưởng người dùng thật. Online evaluation chạy sau deploy hoặc trên canary/A-B traffic để theo dõi drift, latency, failure rate và hành vi trên query thực tế. Human review dùng cho case rủi ro cao, disagreement giữa judge và heuristic, feedback tiêu cực, hoặc các thay đổi không thể đánh giá đầy đủ bằng metric tự động. Quy trình phù hợp là offline gate → canary/online monitoring → human review và bổ sung case mới vào benchmark.

---## Part 2 — Core Coding (14:45–15:40)

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
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

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

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

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
