# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 9:15–12:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 9:15–9:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (9:30–9:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời chứa lời chào, câu chuyển ý hoặc hướng dẫn chung không cần được chứng minh bởi context, nhưng mọi thông tin thực tế về sản phẩm và chính sách vẫn được context hỗ trợ. | Câu trả lời bịa giá bán, chính sách bảo hành, thời hạn đổi trả hoặc thông số sản phẩm không có trong tài liệu | Kiểm tra hallucination; cải thiện prompt yêu cầu chỉ trả lời từ context; bổ sung citation; từ chối trả lời khi thiếu bằng chứng. |
| Answer Relevance | Câu trả lời hữu ích nhưng diễn đạt khác từ khóa trong câu hỏi, hoặc bổ sung một ít thông tin liên quan. | Câu trả lời không giải quyết yêu cầu chính, trả lời nhầm sản phẩm hoặc chuyển sang chủ đề khác. | Làm rõ intent; viết lại prompt; loại bỏ nội dung lan man; kiểm tra query understanding hoặc routing. |
| Context Recall | Câu hỏi đơn giản và các chunks hiện có đã đủ để trả lời, dù không bao phủ toàn bộ expected answer dài. | Retriever bỏ sót điều kiện quan trọng như ngoại lệ bảo hành, phí hoàn trả hoặc bước bắt buộc trong quy trình. | Cải thiện query rewriting; tăng top_k; chunking lại tài liệu; bổ sung metadata/filter; kiểm tra corpus có thiếu dữ liệu không.|
| Context Precision | Retriever lấy thêm một số chunks nhiễu, nhưng chunk đúng vẫn nằm ở đầu và generator vẫn tạo câu trả lời chính xác. | Phần lớn chunks không liên quan hoặc bằng chứng đúng nằm quá thấp, khiến model dùng nhầm tài liệu. | Thêm reranking; cải thiện embedding và metadata filter; giảm top_k; loại bỏ tài liệu trùng hoặc lỗi thời. |
| Completeness | Câu trả lời đã bao phủ hầu hết các khía cạnh quan trọng, dù có thể thiếu một số chi tiết nhỏ. | Câu trả lời bỏ sót các phần quan trọng, dẫn đến thông tin không đầy đủ hoặc gây hiểu lầm. | Cải thiện retrieval để lấy đủ context; tăng top_k; kiểm tra chunking; bổ sung metadata/filter; đảm bảo corpus đầy đủ. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Chuẩn bị nhiều cặp câu trả lời A/B cho cùng một tập câu hỏi và chạy judge trong hai conditions: Condition 1 hiển thị A trước B; Condition 2 đảo lại thành B trước A. Các yếu tố khác như model, rubric, prompt và temperature phải được giữ nguyên. Ghi lại lựa chọn và điểm của judge, sau đó tính tỷ lệ answer đứng đầu được chọn và tỷ lệ kết quả bị đảo khi đổi thứ tự. Nếu cùng một answer thường nhận điểm cao hơn khi đứng đầu, hoặc judge thường chọn answer đầu tiên bất kể đó là A hay B, judge có position bias. Nên randomize thứ tự giữa các test case và dùng đủ nhiều mẫu để tránh kết luận từ vài trường hợp ngẫu nhiên.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Rubric phải chấm theo các yêu cầu nội dung cụ thể, không dùng độ dài làm tín hiệu chất lượng. Mỗi dimension như correctness, completeness, relevance và actionability cần có tiêu chuẩn riêng. Completeness được xác định bằng việc answer có bao phủ các key facts bắt buộc hay không, chứ không phải số từ. Rubric cần ghi rõ không cộng điểm cho nội dung dài, ví dụ không cần thiết hoặc cách diễn đạt hoa mỹ; đồng thời trừ điểm nếu answer lặp lại, lan man hoặc chứa thông tin không liên quan. Judge nên chấm từng dimension độc lập trước khi tính điểm tổng. Vì vậy, một câu trả lời ngắn nhưng đúng và đủ có thể đạt điểm cao hơn câu trả lời dài nhưng có nhiều nội dung thừa.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Việc calibrate LLM judge với human labels giúp đảm bảo rằng các đánh giá của model phản ánh đúng quan điểm và tiêu chuẩn của con người. Điều này giúp giảm bias, tăng độ tin cậy và tính công bằng trong quá trình đánh giá, đồng thời cải thiện khả năng áp dụng các kết quả đánh giá vào thực tế.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.70 | Đây là metric quan trọng nhất vì điểm thấp nghĩa là câu trả lời có thông tin không được context hỗ trợ, có nguy cơ hallucination. Block deployment nếu trung bình dưới 0.70. |
| Answer Relevance | 0.60 | Câu trả lời dưới mức này thường không giải quyết đúng ý định của người dùng hoặc chứa nhiều nội dung không liên quan. |
| Completeness | 0.6 | Câu trả lời dưới mức này thường không bao phủ đủ các key facts bắt buộc. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Offline evaluation nên được dùng để kiểm tra chất lượng model trên các tập dữ liệu đã chuẩn bị sẵn, giúp phát hiện lỗi và điều chỉnh trước khi triển khai. Online evaluation (A/B testing) được dùng để đánh giá model trong môi trường thực tế, đo lường hiệu quả thực sự đối với người dùng. Human review cần thiết khi các metric tự động không đủ để đánh giá chất lượng, đặc biệt với các câu trả lời phức tạp hoặc nhạy cảm. Kết hợp cả ba phương pháp giúp đảm bảo chất lượng và giảm rủi ro khi triển khai model.

---

## Part 2 — Core Coding (9:45–10:40)

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

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

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

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
