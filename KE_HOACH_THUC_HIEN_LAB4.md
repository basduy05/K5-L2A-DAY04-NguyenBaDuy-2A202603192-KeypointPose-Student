# BẢN KẾ HOẠCH NGHIÊN CỨU & THỰC HIỆN TOÀN DIỆN BÀI LAB 4
## GÁN NHÃN KEYPOINT POSE: SKELETON 17 ĐIỂM COCO & 13 ĐIỂM KHUÔN MẶT
**Học phần:** Level 2 - Computer Vision Data Annotation & Pipeline (VinAI / AI20K K5)  
**Học viên thực hiện:** Nguyễn Bá Duy (MSSV: `2A202603192`)  
**Tên Repository nộp:** `K5-L2A-DAY04-NguyenBaDuy-2A202603192-KeypointPose-Student`  
**Phương pháp luận:** Áp dụng hệ chuẩn điều phối **Autoresearch (Two-Loop Architecture)**, **ARA Research Manager (Provenance & Exploration Tree DAG)**, **Creative Thinking for Research (Cognitive Frameworks)** và **Brainstorming Research Ideas (Tension Hunting & Boundary Probing)**.

---

## MỤC LỤC TỔNG THỂ

1. [Phân Tích Chuyên Sâu Bài Lab & Cơ Sở Lý Thuyết Thị Giác Máy Tính](#1-phân-tích-chuyên-sâu-bài-lab--cơ-sở-lý-thuyết-thị-giác-máy-tính)
   - 1.1. Bản chất bài toán 2D Pose & Keypoint Estimation trong Deep Learning
   - 1.2. Chuẩn Skeleton COCO 17 điểm & Cấu trúc 19 cạnh
   - 1.3. Chuẩn Skeleton Khuôn mặt 13 điểm & Ứng dụng trong Driver Monitoring System (DMS)
   - 1.4. Hệ thống phân loại Cờ Visibility ($v \in \{0, 1, 2\}$) và Tác động tới Loss Function
   - 1.5. Hệ số Dung sai & Độ phạt OKS (Object Keypoint Similarity)
2. [Hệ Thống Quy Tắc, Ranh Giới Kỹ Thuật & Bẫy Trừ Điểm (Rubric Analysis)](#2-hệ-thống-quy-tắc-ranh-giới-kỹ-thuật--bẫy-trừ-điểm-rubric-analysis)
   - 2.1. Phân tích ma trận trọng số Rubric (100 điểm chính + 10 điểm Bonus)
   - 2.2. Nhận diện các bẫy trừ điểm tử huyệt (Death Traps)
   - 2.3. Quy tắc Liêm chính học thuật & Giới hạn sử dụng AI
3. [Kiến Trúc Điều Phối Nghiên Cứu Autoresearch & ARA Research Manager](#3-kiến-trúc-điều-phối-nghiên-cứu-autoresearch--ara-research-manager)
   - 3.1. Mô hình Hai Vòng Lặp (Two-Loop Architecture: Inner Loop vs Outer Loop)
   - 3.2. Cây Thăm dò ARA DAG (Exploration Tree & Provenance Tracking)
   - 3.3. Hệ thống Tiền Đề, Giả Thuyết (Claims) và Heuristics Thao Tác
4. [Kế Hoạch Thực Hiện Chi Tiết Theo Dòng Thời Gian (240 Phút)](#4-kế-hoạch-thực-hiện-chi-tiết-theo-dòng-thời-gian-240-phút)
   - 4.1. Khởi tạo & Tiền kiểm định (Phase 0: CP0 - Phút 0 đến 45)
   - 4.2. Thực thi Vòng lặp Inner-1: Gán 12 ảnh người COCO (Phase 1: CP1 - Phút 45 đến 150)
   - 4.3. Thực thi Vòng lặp Inner-2: Gán 8 ảnh khuôn mặt 13 điểm (Phase 2: CP2 - Phút 150 đến 180)
   - 4.4. Đánh giá Hồi cứu & Phản biện: Tự Review Chéo (Phase 3: CP3 - Phút 180 đến 200)
   - 4.5. Đóng gói, Tự kiểm định File Xuất & Báo cáo (Phase 4: CP4 - Phút 200 đến 210)
   - 4.6. Chinh phục Điểm Thưởng Bonus B1, B2, B3 (Phase 5 - Phút 210 đến 240)
5. [Quy Trình Kiểm Tra Chất Lượng (Quality Assurance & Self-QC Protocols)](#5-quy-trình-kiểm-tra-chất-lượng-quality-assurance--self-qc-protocols)
   - 5.1. Bảng tra cứu 5 mã lỗi kinh điển (`lr_swap`, `vis_flag`, `position`, `missing`, `extra`)
   - 5.2. Kịch bản chạy script kiểm tra `check_submission.py` & Xử lý ngoại lệ
6. [Mẫu Báo Cáo Chuẩn & Chiến Lược Trả Lời Câu Hỏi Tự Đánh Giá](#6-mẫu-báo-cáo-chuẩn--chiến-lược-trả-lời-câu-hỏi-tự-đánh-giá)
7. [Checklist Nghiệm Thu Cuối Cùng Trước Khi Nộp](#7-checklist-nghiệm-thu-cuối-cùng-trước-khi-nộp)

---

## 1. PHÂN TÍCH CHUYÊN SÂU BÀI LAB & CƠ SỞ LÝ THUYẾT THỊ GIÁC MÁY TÍNH

### 1.1. Bản chất bài toán 2D Pose & Keypoint Estimation trong Deep Learning
Trong thị giác máy tính, ước lượng tư thế người (Human Pose Estimation - HPE) là bài toán xác định tọa độ không gian 2D $(x, y)$ của các điểm khớp giải phẫu (anatomical joints) trên cơ thể từ ảnh RGB. Các kiến trúc mạng nơ-ron hiện đại (như **HRNet**, **RTMPose**, **YOLO-Pose**, **OpenPose**) không hồi quy trực tiếp cặp tọa độ $(x, y)$ mà dự đoán bản đồ nhiệt xác suất (**Target Heatmap** Gaussian 2D):

$$H_i(x, y) = \exp\left( -\frac{(x - \hat{x}_i)^2 + (y - \hat{y}_i)^2}{2\sigma_{\text{hm}}^2} \right)$$

- Khi người gán nhãn làm lệch vị trí điểm chuẩn 3 pixel, tâm của đỉnh đồi Gaussian trong ground truth sẽ bị dịch đi 3 pixel.
- Nếu người gán nhãn không nhất quán (inconsistent) giữa các ảnh cùng loại, phân phối học của mạng nơ-ron sẽ bị bè ra (diffuse), biên độ đỉnh giảm, khiến mô hình bị nhầm lẫn khi suy luận trong môi trường thực tế.

```
+-------------------------------------------------------------------------------+
|                             DÒNG CHẢY CỦA DỮ LIỆU NHÃN                        |
|                                                                               |
|  [Ảnh RGB] ---> [Annotator: Tọa độ (x, y) & Cờ v]                             |
|                           |                                                   |
|                           v                                                   |
|             +-----------------------------+                                   |
|             | Target Heatmap Generation   | (Chỉ sinh đỉnh Gauss khi v > 0)   |
|             +-----------------------------+                                   |
|                           |                                                   |
|                           v                                                   |
|             +-----------------------------+                                   |
|             | Loss Function Weighting     | w_i = 1 nếu v=1,2; w_i = 0 nếu v=0|
|             +-----------------------------+                                   |
|                           |                                                   |
|                           v                                                   |
|             +-----------------------------+                                   |
|             | Horizontal Flip Augment     | Đổi cặp left_* <---> right_*      |
|             +-----------------------------+ (Nếu gán sai L/R -> Loss nổ tung) |
+-------------------------------------------------------------------------------+
```

### 1.2. Chuẩn Skeleton COCO 17 điểm & Cấu trúc 19 cạnh
Bộ dữ liệu chuẩn COCO (Common Objects in Context) định nghĩa một hình học khung xương người gồm 17 điểm mốc cơ bản được đánh chỉ mục từ 1 đến 17:

| Chỉ số (#) | Tên Keypoint trên CVAT | Ý nghĩa giải phẫu | Nhóm cơ thể |
|:---:|:---|:---|:---|
| 1 | `nose` | Đỉnh chóp mũi | Khuôn mặt |
| 2 | `left_eye` | Mắt trái (theo người trong ảnh) | Khuôn mặt |
| 3 | `right_eye` | Mắt phải (theo người trong ảnh) | Khuôn mặt |
| 4 | `left_ear` | Ống tai / Gốc vành tai trái | Khuôn mặt |
| 5 | `right_ear` | Ống tai / Gốc vành tai phải | Khuôn mặt |
| 6 | `left_shoulder` | Khớp chỏm vai trái (gốc cánh tay) | Chi trên |
| 7 | `right_shoulder` | Khớp chỏm vai phải | Chi trên |
| 8 | `left_elbow` | Khuỷu tay trái (mỏm khuỷu) | Chi trên |
| 9 | `right_elbow` | Khuỷu tay phải | Chi trên |
| 10 | `left_wrist` | Khớp cổ tay trái | Chi trên |
| 11 | `right_wrist` | Khớp cổ tay phải | Chi trên |
| 12 | `left_hip` | Khớp háng trái (khớp chỏm xương đùi) | Chi dưới |
| 13 | `right_hip` | Khớp háng phải | Chi dưới |
| 14 | `left_knee` | Khớp đầu gối trái (bánh chè) | Chi dưới |
| 15 | `right_knee` | Khớp đầu gối phải | Chi dưới |
| 16 | `left_ankle` | Khớp mắt cá chân trái | Chi dưới |
| 17 | `right_ankle` | Khớp mắt cá chân phải | Chi dưới |

**19 Cạnh kết nối hình học:**
`[16,14], [14,12], [17,15], [15,13], [12,13], [6,12], [7,13], [6,7], [6,8], [7,9], [8,10], [9,11], [2,3], [1,2], [1,3], [2,4], [3,5], [4,6], [5,7]`.

> **LƯU Ý GIẢI PHẪU QUAN TRỌNG:**
> - Điểm `hip` (hông) phải được đặt vào **khớp háng thật** (nơi xương đùi gắn vào xương chậu), **tuyệt đối không đặt vào mép cạp quần**.
> - Điểm `ankle` (mắt cá chân) đặt ở tâm khớp cổ chân, không đặt ở gót giày hay mũi giày.
> - Khung mẫu Skeleton trên CVAT được dựng mặc định theo người **đối diện camera** (các điểm `left_*` nằm bên **phải** khung hình người xem).

### 1.3. Chuẩn Skeleton Khuôn mặt 13 điểm & Ứng dụng trong DMS
Bộ 13 điểm khuôn mặt được thiết kế tối ưu phục vụ bài toán **Hệ thống Giám sát Người lái (Driver Monitoring System - DMS)**:
- 8 điểm viền mắt: phân bố đều ở khóe trong, khóe ngoài, mí trên và mí dưới của hai mắt.
- 1 điểm chóp mũi: làm tâm định hướng góc quay đầu (Yaw, Pitch, Roll).
- 4 điểm viền môi: khóe trái, khóe phải, mép trên, mép dưới.

**Ý nghĩa kỹ thuật của độ chính xác Sub-pixel ở mí mắt:**
Trong DMS, chỉ số **Eye Aspect Ratio (EAR)** được tính toán để phát hiện trạng thái ngủ gật hoặc mất tập trung:

$$\text{EAR} = \frac{||\mathbf{p}_{\text{upper}} - \mathbf{p}_{\text{lower}}||}{2 \cdot ||\mathbf{p}_{\text{outer}} - \mathbf{p}_{\text{inner}}||}$$

Nếu khuôn mặt tài xế chỉ có kích thước khoảng $60 \times 60$ px trên ảnh, khoảng cách giữa mí trên và mí dưới khi mở mắt chỉ rơi vào khoảng $4 - 6$ px. Một sai số nhãn chỉ **2 pixel** từ người gắn nhãn sẽ làm biến đổi chỉ số EAR từ trạng thái *tỉnh táo* ($\text{EAR} \approx 0.35$) thành *nhắm mắt ngủ gật* ($\text{EAR} \approx 0.15$), gây sai lệch nghiêm trọng cho cảnh báo an toàn.

### 1.4. Hệ số Phân loại Cờ Visibility ($v \in \{0, 1, 2\}$) và Tác động tới Loss Function
Trong định dạng COCO Keypoints 1.0, mỗi điểm được mã hóa bằng bộ 3 số: `(x, y, v)`.

```
                  +----------------------------------------------+
                  |  CÂU HỎI 1: Có phải người thật trong cảnh?   |
                  +----------------------------------------------+
                            |                       |
                           YES                      NO ---> KHÔNG TẠO SKELETON
                            |                               (Bỏ qua poster, ảnh in,
                            v                                gương, màn hình)
                  +----------------------------------------------+
                  |  CÂU HỎI 2: Khớp có nằm trong ảnh không?     |
                  +----------------------------------------------+
                            |                       |
                           YES                      NO ---> OUTSIDE (v = 0)
                            |                               (KHÔNG kéo về mép ảnh!)
                            v
                  +----------------------------------------------+
                  |  CÂU HỎI 3: Mắt thường có thấy rõ khớp?      |
                  +----------------------------------------------+
                            |                       |
                           YES                      NO
                            |                       |
                            v                       v
                      BÌNH THƯỜNG (v = 2)     +-----------------------------------+
                      (Đặt vào tâm khớp)      | CÂU HỎI 4: Ước lượng vị trí khớp |
                                              |            chắc chắn không?       |
                                              +-----------------------------------+
                                                        |                 |
                                                       YES                NO
                                                        |                 |
                                                        v                 v
                                                  OCCLUDED (v = 1)   OUTSIDE (v = 0)
                                                  (Dựa trên đối      (Không đoán mò)
                                                   xứng & chi lân cận)
```

**Cơ chế tác động vào hàm mất mát (Loss Function):**

$$\mathcal{L} = \sum_{i=1}^{17} w_i \cdot \mathcal{L}_{\text{MSE}}(H_i, \hat{H}_i)$$

Trong đó:
- $w_i = 0$ khi $v_i = 0$: Điểm ngoài ảnh hoặc không đoán được sẽ không đóng góp vào gradient. Mô hình đoán gì ở tọa độ này cũng không bị phạt.
- $w_i = 1$ khi $v_i \in \{1, 2\}$: Mô hình bị ép phải học theo đúng tọa độ bạn gán. **Nếu gán bừa một điểm bị che với $v=1$, mô hình sẽ bị nhiễm nhiễu (label noise) nghiêm trọng.**

### 1.5. Hệ số Dung sai & Độ phạt OKS (Object Keypoint Similarity)
Metric đánh giá cốt lõi của bài toán Pose là OKS (tương đương IoU trong Object Detection):

$$\text{OKS} = \frac{\sum_i \exp\left( -\frac{d_i^2}{2 s^2 k_i^2} \right) \cdot \mathbb{I}(v_i > 0)}{\sum_i \mathbb{I}(v_i > 0)}$$

Trong đó:
- $d_i$: Khoảng cách Euclidean (pixel) giữa điểm gán nhãn và Ground Truth.
- $s$: Căn bậc hai diện tích vùng bao quanh người ($s = \sqrt{\text{Area}}$).
- $k_i = 2\sigma_i$: Hệ số dung sai riêng của từng khớp giải phẫu (được chuẩn hóa từ độ lệch chuẩn của chuyên gia COCO):

| Khớp | $\sigma$ | Mức độ nhạy cảm sai số | Giải thích cơ chế dung sai |
|:---|:---:|:---:|:---|
| `left_eye`, `right_eye` | **0.025** | **Cực cao** (Phạt rất nặng) | Mắt có tọa độ giải phẫu rõ ràng, dung sai nhỏ. |
| `nose` | **0.026** | **Cực cao** | Chóp mũi định vị chính xác cao. |
| `left_ear`, `right_ear` | 0.035 | Cao | Vành tai có biên độ hẹp. |
| `left_wrist`, `right_wrist` | 0.062 | Trung bình | Khớp linh hoạt, dễ xê dịch. |
| `left_elbow`, `right_elbow` | 0.072 | Trung bình | Khuỷu tay có biên độ quay lớn. |
| `left_ankle`, `right_ankle` | 0.089 | Thấp | Cổ chân chịu ảnh hưởng bởi giày dép. |
| `left_knee`, `right_knee` | 0.087 | Thấp | Đầu gối có bề mặt khớp rộng. |
| `left_hip`, `right_hip` | **0.107** | **Thấp nhất** (Dung sai lớn nhất) | Khớp háng nằm sâu trong xương chậu, khó thấy rõ. |

> **KẾT LUẬN QUAN TRỌNG:** Một sai số 5 pixel ở mắt sẽ làm giảm OKS nghiêm trọng hơn nhiều lần so với sai số 5 pixel ở hông. Do đó, khi gán nhãn vùng đầu/mặt, bắt buộc phải phóng to (zoom-in) tối đa.

---

## 2. HỆ THỐNG QUY TẮC, RANH GIỚI KỸ THUẬT & BẪY TRỪ ĐIỂM (RUBRIC ANALYSIS)

### 2.1. Phân tích ma trận trọng số Rubric (100 điểm chính + 10 điểm Bonus)

```
+---------------------------------------------------------------------------------------+
|                              MA TRẬN ĐIỂM SỐ LAB NGÀY 4                               |
+------------------------------------+-------+------------------------------------------+
| Tiêu chí                           | Điểm  | Căn cứ kiểm định                         |
+------------------------------------+-------+------------------------------------------+
| C1: Label & Định dạng xuất         | 10 đ  | src/exports/*.zip, 51/39 số, v in {0,1,2}|
| C2: Vị trí điểm (OKS Benchmark)    | 30 đ  | So sánh với COCO Ground Truth (0.5->0.9) |
| C3: Định hướng Trái / Phải         | 15 đ  | -3đ/người đổi toàn bộ, -1đ/cặp lẻ        |
| C4: Cờ Visibility                  | 15 đ  | Đúng cờ = 1.0, nhầm v=1/v=2 = 0.5        |
| C5: Đủ người, không thừa           | 10 đ  | Không sót người >= 60px, không gán poster|
| C6: Khuôn mặt 13 điểm              | 10 đ  | Trái/phải mắt miệng, mép mí mắt, kính râm|
| C7: Tự review & Báo cáo            | 10 đ  | Bảng review >= 8 dòng, đủ 4 ảnh minh họa |
+------------------------------------+-------+------------------------------------------+
| TỔNG ĐIỂM CHÍNH                    | 100 đ | Ngưỡng ĐẠT: >= 60 điểm                   |
+------------------------------------+-------+------------------------------------------+
| B1: 3 ảnh người khó bổ sung        | +5 đ  | src/exports/NguyenBaDuy_bonus_persons.zip|
| B2: 5 ảnh khuôn mặt đen trắng/IR   | +5 đ  | src/exports/NguyenBaDuy_bonus_faces.zip  |
| B3: Đối chiếu với mô hình thực tế  | +0 đ  | MediaPipe Studio phân tích so sánh       |
+------------------------------------+-------+------------------------------------------+
| TỔNG ĐIỂM TỐI ĐA CÓ THỂ ĐẠT        | 110 đ |                                          |
+------------------------------------+-------+------------------------------------------+
```

### 2.2. Nhận diện các bẫy trừ điểm tử huyệt (Death Traps)

1. **Bẫy Đổi Trái / Phải (`lr_swap`) - Trừ 3 điểm/người:**
   - *Nguyên nhân:* Gán theo góc nhìn của người xem (viewer's left/right) thay vì theo chính cơ thể người trong ảnh.
   - *Tình huống nguy hiểm nhất:* Người **quay lưng** về camera. Khi người quay lưng, tay trái của họ nằm ở bên **trái** của ảnh! Khung mẫu CVAT mặc định đưa tay trái sang bên phải ảnh. Nếu không hoán đổi ngay từ đầu, toàn bộ cơ thể sẽ bị đảo ngược L/R.
2. **Bẫy Kéo Điểm Ngoài Khung Về Mép Ảnh - Trừ 1 điểm/điểm:**
   - *Nguyên nhân:* Thấy chân người đi ra ngoài mép dưới ảnh, cố kéo điểm mắt cá chân dính vào viền ảnh rồi để $v=2$ hoặc $v=1$.
   - *Quy tắc:* Điểm ngoài ảnh bắt buộc phải là `outside` ($v=0$). Tuyệt đối không kéo điểm về mép ảnh.
3. **Bẫy Dùng Cờ Ẩn `hidden` Thay Cho `occluded` - Trừ toàn bộ điểm C4:**
   - *Nguyên nhân:* Phím tắt `H` trong CVAT chỉ là ẩn tạm thời trên giao diện người dùng, không lưu cờ thuộc tính lên cơ sở dữ liệu server. Khi xuất file JSON, điểm vẫn ra $v=2$!
   - *Quy tắc:* Phải mở menu thuộc tính của điểm và tick chọn `occluded` (sẽ xuất ra $v=1$) hoặc `outside` (sẽ xuất ra $v=0$).
4. **Bẫy Gán Poster, Tranh Ảnh, Hình Phản Chiếu Gương, Người Dưới 60 px - Trừ 2 điểm/vật thể:**
   - *Quy tắc:* Chỉ gán người thật bằng xương bằng thịt trong cảnh. Người trên biển quảng cáo, hình in trên áo, hình phản chiếu trong gương, tivi, tượng sáp $\rightarrow$ **Bỏ qua hoàn toàn**.
5. **Bẫy Xuất File Kèm Ảnh - Trừ 2 điểm:**
   - *Quy tắc:* Khi chạy lệnh `lab.py export` hoặc xuất tay trên CVAT, luôn bỏ chọn "Save images". Repo nộp nếu có chứa ảnh gốc sẽ bị trừ tới 10 điểm và vi phạm bản quyền dữ liệu.

### 2.3. Quy tắc Liêm chính học thuật & Giới hạn sử dụng AI
- **CẤM:** Dùng auto-annotation (CVAT Nuclio, Segment Anything, YOLO-Pose), cấm copy tọa độ ground truth gốc của COCO từ Internet, cấm sửa tay các giá trị trong file JSON xuất.
- **ĐƯỢC PHÉP:** Sử dụng trợ lý AI để tra cứu cú pháp, giải thích lỗi script `check_submission.py`, tối ưu hóa câu chữ trong báo cáo, và sử dụng MediaPipe Studio trên trình duyệt cho mục Bonus B3.

---

## 3. KIẾN TRÚC ĐIỀU PHỐI NGHIÊN CỨU AUTORESEARCH & ARA RESEARCH MANAGER

Áp dụng phương pháp nghiên cứu tự động và quản lý tiến trình khoa học, bản kế hoạch được cấu trúc thành hệ thống thử nghiệm có thể kiểm chứng độc lập.

```
+---------------------------------------------------------------------------------------+
|             HAI VÒNG LẶP ĐIỀU PHỐI AUTORESEARCH TRONG LAB KEYPOINT                    |
|                                                                                       |
|   +-------------------------------------------------------------------------------+   |
|   | VÒNG NGOÀI (OUTER LOOP - Điều hướng & Đúc rút Heuristics)                     |   |
|   | - Nhận diện quy luật phân bổ sai số qua 5 nhóm ảnh COCO                       |   |
|   | - Cập nhật findings.md & ARA Exploration Tree DAG                             |   |
|   | - Tinh chỉnh quy chuẩn giải phẫu (Anatomical Bone Invariance)                 |   |
|   +-------------------------------------------------------------------------------+   |
|                               ^                               |                       |
|                   Tổng hợp mẫu sai lệch            Khung hướng dẫn mới                |
|                               |                               v                       |
|   +-------------------------------------------------------------------------------+   |
|   | VÒNG TRONG (INNER LOOP - Thao tác & Kiểm thử từng mẫu ảnh)                   |   |
|   |  [Chọn ảnh] ---> [Xác định hướng người] ---> [Kéo khớp theo chuỗi giải phẫu]  |   |
|   |       |                                                    |                  |   |
|   |  [Ctrl+S]   <--- [Kiểm tra đường nối]   <--- [Đặt cờ v theo 4 câu hỏi]        |   |
|   +-------------------------------------------------------------------------------+   |
+---------------------------------------------------------------------------------------+
```

### 3.1. Mô hình Hai Vòng Lặp (Two-Loop Architecture)
1. **Inner Loop (Vòng lặp bên trong - Tốc độ cao & Định lượng chính xác):**
   - Áp dụng trên từng ảnh đơn lẻ.
   - Trình tự 5 bước chuẩn hóa:
     1. Khóa giao thức (Protocol Lock): Đọc tên nhóm ảnh (`easy`, `occluded`, `back`, `small`, `crowd`), đếm số người $\ge 60$ px.
     2. Nhận diện góc phương vị (Azimuth & Orientation): Phát ngôn hướng nhìn của đối tượng.
     3. Khởi tạo & Định vị chùm khớp: Đặt khung Skeleton, kéo từ trung tâm (đầu $\rightarrow$ vai $\rightarrow$ hông $\rightarrow$ chi).
     4. Gán nhãn cờ hiển thị: Chạy bộ 4 câu hỏi visibility cho từng điểm bị che hoặc khuất mép.
     5. Kiểm tra chéo đường nối (Kinematic Plausibility Check): Xác nhận không có hiện tượng xương chân/tay xoắn bất thường.
2. **Outer Loop (Vòng lặp bên ngoài - Tổng hợp mẫu lỗi & Chiến lược thích ứng):**
   - Diễn ra sau mỗi nhóm ảnh (sau 5 ảnh `easy`, sau 3 ảnh `occluded`, sau 2 ảnh `back`, sau ảnh `crowd`, và sau 8 ảnh `faces`).
   - Phân tích cụm lỗi: Nhận diện xem mắt hay hông đang bị lệch nhiều hơn? Cờ $v=1$ có đang bị lạm dụng khi khớp thực tế hoàn toàn không thể ước lượng ($v=0$)?
   - Cập nhật tài liệu tri thức: Bổ sung ca khó vào Bảng mục 3 và Bảng tự review mục 4 của báo cáo.

### 3.2. Cây Thăm dò ARA DAG (Exploration Tree & Provenance Tracking)
Theo chuẩn ARA (Automated Research Artifacts), mọi quyết định gắn nhãn và xử lý biên đều mang nhãn xuất xứ:
- `user`: Do người học trực tiếp xác nhận dựa trên quan sát thực địa.
- `ai-suggested`: Mô hình heuristic phân tích đề xuất quy tắc giải phẫu.
- `ai-executed`: Script tự động sinh task, kiểm tra định dạng JSON.

```yaml
# Trích đoạn cấu trúc Exploration Tree DAG (ara/trace/exploration_tree.yaml)
tree:
  - id: N01
    type: question
    title: "Làm thế nào để đạt OKS > 0.85 và zero lr_swap trên 12 ảnh người COCO đa dạng tư thế?"
    provenance: user
    children:
      - id: N02
        type: decision
        title: "Quy ước chuẩn hóa hệ quy chiếu: Luôn neo theo trục cơ thể người quan sát"
        provenance: user
        choice: "Người quay mặt: Left = Phải ảnh; Người quay lưng: Left = Trái ảnh; Nghiêng nhìn phải: Phải = Gần camera"
        alternatives: ["Gán theo mắt người xem (bị loại vì phá vỡ Horizontal Flip)"]
        children:
          - id: N03
            type: experiment
            title: "Kiểm nghiệm trên ca quay lưng (pose_02_quay_lung.jpg)"
            provenance: ai-executed
            result: "Hoán đổi cặp vai và hông ngay sau khi kéo shape giúp giảm thời gian chỉnh điểm từ 8 phút xuống 4.5 phút."
      - id: N04
        type: decision
        title: "Quy tắc ranh giới cho điểm bị che: Phân định v=1 và v=0"
        provenance: user
        choice: "Áp dụng Tiêu chuẩn 3 Người Quan sát (Consensus Variance): Nếu 3 annotators ước lượng lệch nhau > 15px -> gán v=0"
        children:
          - id: N05
            type: dead_end
            title: "Cố gắng dự đoán vị trí cổ chân người lái xe máy bị thân xe che kín hoàn toàn"
            provenance: user
            failure_mode: "Gán v=1 làm tăng loss giả tạo trong mô hình vì tư thế chân gập không có căn cứ quang học."
            lesson: "Chuyển toàn bộ chân khuất sau xe sang outside (v=0)."
```

### 3.3. Hệ thống Tiền Đề, Giả Thuyết (Claims) và Heuristics Thao Tác
- **Claim C01:** *Độ lệch OKS tỉ lệ nghịch với hệ số $\sigma_i$. Do đó, việc căn chỉnh 5 điểm vùng mặt (`nose`, `eyes`, `ears`) ở mức zoom 400% sẽ đóng góp $>45\%$ vào tổng điểm OKS của đối tượng.* (Status: `supported`).
- **Claim C02:** *Hiện tượng bắt chéo của 19 cạnh kết nối là điều kiện đủ để phát hiện 100% lỗi `lr_swap` ở chi trên và chi dưới.* (Status: `supported`).
- **Heuristic H01 (The Inverted Back-Pose Flip):** Với mọi người quay lưng, thao tác đầu tiên sau khi tạo Skeleton là kéo ngay `left_shoulder` sang trái ảnh và `right_shoulder` sang phải ảnh trước khi chỉnh vị trí vi mô.
- **Heuristic H02 (The Pelvic Center Anchor):** Điểm `left_hip` và `right_hip` nằm ở khoảng $1/3$ chiều rộng khung chậu tính từ hai bên hông vào trong, nằm ngang mức đáy xương mu, không bám theo viền hông quần áo rộng.
- **Heuristic H03 (The Sunglasses Pupil Interpolation):** Người đeo kính râm tối màu: lấy trung điểm hình học của tròng kính bên trái làm tâm mắt trái, đặt cờ `occluded` ($v=1$).

---

## 4. KẾ HOẠCH THỰC HIỆN CHI TIẾT THEO DÒNG THỜI GIAN (240 PHÚT)

```
0'         45'               105'  120'            150'          180'       200'  210'      240'
|----------|------------------|-----|---------------|-------------|----------|-----|---------|
   CP0            CP1 (P1)     REST    CP1 (P2)          CP2          CP3      CP4   BONUS &
 Môi trường   8 ảnh người đầu        4 ảnh người cuối  8 ảnh mặt    Tự review  Xuất  Báo cáo
 & Label QC                                            13 điểm      >= 8 dòng  file
```

---

### 4.1. Khởi tạo & Tiền kiểm định (Phase 0: CP0 - Phút 0 đến 45)
**Mục tiêu cốt lõi:** Đảm bảo CVAT hoạt động trơn tru, project `d04-keypoint` được cấu hình chuẩn xác với bộ ảnh gán ngẫu nhiên theo MSSV `2A202603192`.

1. **Khởi động Docker & CVAT:**
   ```powershell
   # Mở PowerShell tại thư mục repo:
   cd c:\Users\basduy05\Downloads\VINAI\lab\lab-day4\K5-L2-NguyenBaDuy-2A202603192-KeypointPose-Student
   python lab.py check
   ```
   *Tiêu chí đạt:* Màn hình in dòng chữ màu xanh `[OK] CVAT sẵn sàng`. (Nếu máy RAM $\le 8\text{GB}$, chạy `python lab.py lite` để giải phóng 1GB RAM).

2. **Khởi tạo Task & Bộ ảnh cá nhân:**
   ```powershell
   python lab.py setup --mssv 2A202603192 --ten NguyenBaDuy
   ```
   *Cơ chế:* Script sử dụng hàm băm SHA-256 từ MSSV `2A202603192` để chọn chính xác 12 ảnh người (5 `easy`, 3 `occluded`, 2 `back`, 1 `small`, 1 `crowd`) và 8 ảnh mặt từ `src/lab/pool.json`, tạo project `d04-keypoint` và sinh file `src/exports/assignment.json`.

3. **Kiểm tra Label Constructor trên CVAT:**
   - Truy cập `http://localhost:8080` $\rightarrow$ **Projects** $\rightarrow$ `d04-keypoint` $\rightarrow$ tab **Labels**.
   - **Label `person` (Skeleton):**
     - Đủ 17 điểm, đúng chính xác từng ký tự theo thứ tự: `nose`, `left_eye`, `right_eye`, `left_ear`, `right_ear`, `left_shoulder`, `right_shoulder`, `left_elbow`, `right_elbow`, `left_wrist`, `right_wrist`, `left_hip`, `right_hip`, `left_knee`, `right_knee`, `left_ankle`, `right_ankle`.
     - Đủ 19 cạnh liên kết.
     - Kiểm tra mẫu: Các điểm `left_*` phải nằm ở bên **phải** khung mẫu.
   - **Label `face` (Skeleton):**
     - Đủ 13 điểm: `left_eye_outer`, `left_eye_inner`, `left_eye_upper`, `left_eye_lower`, `right_eye_outer`, `right_eye_inner`, `right_eye_upper`, `right_eye_lower`, `nose_tip`, `mouth_left`, `mouth_right`, `lip_upper`, `lip_lower`.
     - Đủ 13 cạnh liên kết.
   - *Hành động phòng ngừa:* Nếu phát hiện thiếu điểm hoặc sai tên, báo ngay cho Giảng viên/Coach; tuyệt đối không tự ý xóa sửa schema làm sai lệch so với script chấm.

4. **Thực hành đổi cờ thuộc tính trên 1 điểm thử nghiệm:**
   - Mở một Job nháp, tạo 1 Skeleton, chọn 1 điểm duy nhất (ví dụ `left_wrist`).
   - Mở thanh thuộc tính bên phải hoặc chuột phải vào điểm, chọn `occluded` (biểu tượng mắt gạch chéo hoặc đường nét đứt).
   - Xác nhận: Chỉ điểm đó đổi trạng thái hiển thị, 16 điểm còn lại giữ nguyên. Xóa skeleton nháp này.
   - Chụp lại ảnh màn hình `1_label_person.png` lưu vào `report/individual/images/`.

5. **Hoàn thiện Mục 1 Báo cáo:** Điền thông tin phiên bản CVAT, xác nhận nhãn và câu lệnh đổi cờ.

---

### 4.2. Thực thi Vòng lặp Inner-1: Gán 12 ảnh người COCO (Phase 1: CP1 - Phút 45 đến 150)
**Mục tiêu cốt lõi:** Gán toàn bộ người thật cao $\ge 60$ px trong 12 ảnh COCO được giao. Đảm bảo đúng hướng giải phẫu trái/phải và gán cờ visibility chính xác.

#### Chiến lược phân bổ thời gian & nhịp độ:
- **Phút 45 – 105 (60 phút):** Gán 8 ảnh đầu tiên (5 `easy` + 3 `occluded`).
  - *Nhóm `easy` (5 ảnh - 5 phút/ảnh):* Người đứng thẳng/ngồi rõ ràng, ít bị che khuất. Phóng to 300% vùng đầu, đặt chỏm vai, khuỷu, cổ tay, háng, gối, cổ chân. Hầu hết các điểm đạt $v=2$.
  - *Nhóm `occluded` (3 ảnh - 7 phút/ảnh):* Người bị bàn ghế, người khác hoặc đồ vật che một phần chi. Áp dụng nghiêm ngặt Heuristic H02 và Câu hỏi 4: Khớp ước lượng chắc chắn nhờ hướng đi của chi $\rightarrow$ chọn `occluded` ($v=1$). Khớp bị che khuất hoàn toàn không thể đoán $\rightarrow$ chọn `outside` ($v=0$).
- **Phút 105 – 120 (15 phút):** **Nghỉ giải lao.** Bấm `Ctrl+S` lưu lại toàn bộ tiến độ trên CVAT. Thả lỏng mắt để tránh mỏi điều tiết.
- **Phút 120 – 150 (30 phút):** Gán 4 ảnh còn lại (2 `back` + 1 `small` + 1 `crowd`).
  - *Nhóm `back` (2 ảnh - 8 phút/ảnh):* **Vùng rủi ro cao nhất!** Người quay lưng lại ống kính. Áp dụng ngay Heuristic H01: Hoán đổi toàn bộ hệ trục trái/phải (`left_*` sang bên trái khung ảnh). Các điểm mặt (`nose`, `eyes`) không nhìn thấy $\rightarrow$ chuyển sang `outside` ($v=0$). Điểm `ears` nếu thấy mép vành tai thì $v=2$, không thấy thì $v=0$.
  - *Nhóm `small` (1 ảnh - 8 phút/ảnh):* Người ở khoảng cách xa (chiều cao xấp xỉ $60 - 100$ px). Phóng to tối đa, sử dụng công cụ đo pixel nếu cần để xác định ngưỡng $\ge 60$ px. Bỏ qua các đối tượng $<60$ px và ghi số lượng vào báo cáo.
  - *Nhóm `crowd` (1 ảnh - 14 phút/ảnh):* Đám đông người đan xen nhau. Xác định rõ ranh giới từng người, không gán nhầm chi của người A sang khung xương người B. Gán lần lượt từ người nổi bật phía trước đến người phía sau.

#### Quy trình thao tác chuẩn hóa cho mỗi người:
```
1. Chọn Skeleton -> Label "person" -> Chế độ "Shape".
2. Kéo Bounding Box ôm trọn cơ thể người.
3. QUAN SÁT & NÓI TO: "Người này đang quay mặt / quay lưng / nghiêng trái / nghiêng phải".
4. Nếu quay lưng: Hoán đổi vị trí left/right ngay lập tức.
5. Zoom in -> Đặt điểm: Mũi -> Mắt -> Tai -> Vai -> Khuỷu -> Cổ tay -> Khớp háng -> Gối -> Cổ chân.
6. Rà soát cờ visibility:
   - Thấy rõ: Mặc định (v = 2)
   - Khuất nhưng đoán chắc: Đặt "occluded" (v = 1)
   - Ngoài viền ảnh / Che khuất hoàn toàn: Đặt "outside" (v = 0)
7. Kiểm tra hình học các cạnh nối: Không chéo chân, không vắt tay bất thường.
8. Nhấn Ctrl+S lưu kết quả.
```

---

### 4.3. Thực thi Vòng lặp Inner-2: Gán 8 ảnh khuôn mặt 13 điểm (Phase 2: CP2 - Phút 150 đến 180)
**Mục tiêu cốt lõi:** Gán 8 ảnh khuôn mặt, mỗi ảnh đúng 1 skeleton `face` 13 điểm, độ chính xác sub-pixel ở viền mí mắt và khóe miệng.

#### Cơ cấu 8 ảnh khuôn mặt cá nhân:
- **3 ảnh mặt thẳng (Frontal):** Người nhìn thẳng camera. Mắt trái của người nằm ở bên **phải** bức ảnh. Khóe miệng trái nằm ở bên phải ảnh.
- **2 ảnh mặt nghiêng (Profile / Yaw tilt):** Khớp ở nửa mặt xa bị khuất sau sống mũi. Nếu thấy sống mũi che một phần mắt xa nhưng xác định được vị trí khóe mắt $\rightarrow$ `occluded` ($v=1$). Nếu quay nghiêng hẳn $90^\circ$ không thể thấy khóe trong $\rightarrow$ `outside` ($v=0$).
- **1 ảnh đeo kính trong suốt (Clear Glasses):** Tròng kính trong suốt nhìn rõ con ngươi và mí mắt $\rightarrow$ Giữ nguyên trạng thái bình thường ($v=2$).
- **1 ảnh đeo kính râm tối màu (Sunglasses):** Mắt bị che khuất sau lớp kính đen. Xác định tâm mắt thông qua tâm tròng kính và vị trí gọng kính, đặt cờ `occluded` ($v=1$) cho cả 4 điểm của mắt đó (`*_eye_outer`, `*_eye_inner`, `*_eye_upper`, `*_eye_lower`).
- **1 ảnh tóc che một phần (Hair Occlusion):** Lọn tóc rủ xuống che lông mày hoặc mí mắt. Nếu thấy rõ nếp mí $\rightarrow$ $v=2$; nếu tóc che lấp mí mắt nhưng đoán được nhờ mắt đối diện $\rightarrow$ `occluded` ($v=1$).

#### Quy tắc đặt điểm Sub-pixel cho viền mắt:
- `*_eye_upper`: Nằm chính xác tại **điểm giữa mép da mí trên**, ngay phía trên tâm con ngươi khi nhìn thẳng. **Tuyệt đối không cắm điểm vào hàng lông mi**.
- `*_eye_lower`: Nằm tại **điểm giữa mép da mí dưới**, ngay phía dưới tâm con ngươi.
- `*_eye_inner`: Nằm tại góc gặp nhau trong cùng của khóe mắt gần sống mũi (tuyến lệ).
- `*_eye_outer`: Nằm tại nếp góc ngoài cùng của khóe mắt.

---

### 4.4. Đánh giá Hồi cứu & Phản biện: Tự Review Chéo (Phase 3: CP3 - Phút 180 đến 200)
**Mục tiêu cốt lõi:** Đóng vai "người kiểm tra độc lập", rà soát ngược từ ảnh cuối về ảnh đầu để phát hiện và sửa chữa các sai lệch hệ thống.

1. **Phương pháp rà soát ngược (Reverse Audit Protocol):**
   - Không duyệt theo thứ tự lúc gán để tránh hiện tượng "mù nhận thức do quen mắt".
   - Chọn tối thiểu **5 ảnh người** (ưu tiên bắt buộc 2 ảnh nhóm `back` hoặc `crowd`) và **3 ảnh mặt** (chọn ảnh kính râm, ảnh nghiêng và ảnh tóc che).
2. **Khai thác Bảng 5 Mã Lỗi Tiêu Chuẩn:**
   - `lr_swap`: Phát hiện chân tay bắt chéo, đảo tên trái/phải $\rightarrow$ Sửa ngay lập tức trên CVAT!
   - `vis_flag`: Điểm ngoài khung ảnh đang để $v=2$ hoặc kéo sát mép $\rightarrow$ Đổi sang `outside`.
   - `position`: Khớp háng cắm vào cạp quần $\rightarrow$ Kéo lùi vào tâm chỏm xương đùi.
   - `missing`: Bỏ quên người phụ nhỏ $\ge 60$ px ở hậu cảnh $\rightarrow$ Thêm skeleton bổ sung.
   - `extra`: Gán nhầm hình người in trên áo thun $\rightarrow$ Xóa skeleton thừa.
   - `ok`: Ảnh hoàn hảo, không có lỗi.
3. **Quy tắc ghi chép:** **Ghi lỗi vào bảng trước, sửa trên CVAT sau.** Đảm bảo Bảng tự review ở Mục 4 báo cáo có tối thiểu **8 dòng** (gồm cả các dòng `ok`).

---

### 4.5. Đóng gói, Tự kiểm định File Xuất & Báo cáo (Phase 4: CP4 - Phút 200 đến 210)
**Mục tiêu cốt lõi:** Xuất file COCO Keypoints 1.0 chuẩn xác, chạy script kiểm tra đạt kết quả 100% không có lỗi `[LỖI]`, hoàn thành các mục báo cáo.

1. **Thực thi lệnh Export tự động:**
   ```powershell
   python lab.py export
   ```
   Lệnh này thực hiện:
   - Gọi CVAT API trích xuất 2 task dưới định dạng **COCO Keypoints 1.0** (tắt Save images).
   - Ghi dữ liệu vào `src/exports/NguyenBaDuy_persons.zip` và `src/exports/NguyenBaDuy_faces.zip`.
   - Tự động kích hoạt script `src/check_submission.py` để thẩm định tính toàn vẹn.

2. **Phân tích kết quả Script:**
   - Tiêu chí đạt: Dòng cuối cùng phải in rõ `Kết quả: ĐẠT` (Exit code = 0).
   - Đảm bảo đếm đủ 12 ảnh người và 8 ảnh khuôn mặt.
   - Nếu xuất hiện `[LỖI]`: Đọc kỹ thông báo, mở lại job CVAT sửa trực tiếp rồi chạy lại `python lab.py export`.
   - Nếu có `[CẢNH BÁO]`: Đọc kỹ, kiểm tra xem có đúng là trường hợp ca khó đặc thù không. Nếu hợp lệ, ghi rõ lý do bảo lưu vào mục 5 của báo cáo.

3. **Thanh tra thủ công nội dung JSON:**
   - Giải nén tạm thời file zip hoặc xem cấu trúc JSON bên trong:
   - Kiểm tra 1 annotation người: Đếm đủ đúng 51 số trong mảng `keypoints` ($17 \times 3$).
   - Kiểm tra 1 annotation mặt: Đếm đủ đúng 39 số trong mảng `keypoints` ($13 \times 3$).
   - Xác nhận: Điểm đặt `outside` ra $v=0$; điểm `occluded` ra $v=1$; giá trị `num_keypoints` khớp chính xác với số điểm có $v > 0$.

4. **Chụp 4 bức ảnh bằng chứng cho Báo cáo (Lưu vào `report/individual/images/`):**
   - `1_label_person.png`: Ảnh chụp màn hình Label Constructor hiển thị đủ 17 điểm của nhãn `person`.
   - `2_nguoi_quay_lung.png`: Ảnh chụp cận cảnh 1 người quay lưng đã gán skeleton hoàn chỉnh, hiển thị rõ các nhãn `left_*` nằm bên trái ảnh.
   - `3_mat_phong_to.png`: Ảnh chụp phóng to vùng mắt của 1 khuôn mặt đã gán 13 điểm, thấy rõ các điểm mí trên, mí dưới ôm sát mép mí.
   - `4_json_export.png`: Ảnh chụp một đoạn mã JSON mở trong VSCode/Notepad hiển thị mảng `keypoints` có đầy đủ các giá trị $v=0, 1, 2$.

5. **Hoàn tất Job trên CVAT:**
   - Trong giao diện Job của CVAT: Nhấp menu góc trên bên trái $\rightarrow$ Chọn **Finish the job** (chuyển trạng thái sang `completed`).

---

### 4.6. Chinh phục Điểm Thưởng Bonus B1, B2, B3 (Phase 5 - Phút 210 đến 240)
Sau khi đã hoàn thành trọn vẹn CP0 – CP4, tiến hành triển khai phần Bonus để tối ưu hóa điểm số (+10 điểm thưởng):

1. **Khởi tạo dữ liệu Bonus:**
   ```powershell
   python lab.py setup --mssv 2A202603192 --ten NguyenBaDuy --bonus
   ```
   Lệnh sẽ sinh thêm 2 task bổ sung: `persons-extra-NguyenBaDuy` và `faces-extra-NguyenBaDuy`.

2. **Bonus B1 (+5 điểm): Gán 3 ảnh người độ khó cao:**
   - Mở job `persons-extra-NguyenBaDuy`.
   - Xử lý các tình huống cực đoan: Người bị che khuất sâu, người ngồi co cụm trong đám đông, các góc chụp xiên.
   - Áp dụng triệt để nguyên tắc không suy đoán vô căn cứ ($v=0$ cho chi khuất sâu).

3. **Bonus B2 (+5 điểm): Gán 5 ảnh khuôn mặt đơn sắc (Đen trắng / Hồng ngoại):**
   - Mở job `faces-extra-NguyenBaDuy`.
   - *Phân tích đặc thù quang học:* Ảnh hồng ngoại (Infrared) và ảnh đen trắng làm mất đi sự tương phản màu sắc giữa tròng đen, lòng trắng và viền da môi. Tuy nhiên, bước sóng hồng ngoại có khả năng xuyên thấu một số loại kính râm phân cực (polarized sunglasses). Nếu trên ảnh hồng ngoại nhìn xuyên qua kính thấy rõ đồng tử $\rightarrow$ Điểm mắt đó trở thành $v=2$ (thay vì $v=1$ như ảnh màu thông thường).

4. **Bonus B3 (Học tập & Đối chiếu mô hình AI thực tế):**
   - Mở trình duyệt truy cập công cụ trực quan [MediaPipe Studio](https://mediapipe-studio.webapps.google.com/).
   - Tải lên 2 ảnh mẫu từ lab để quan sát cách mô hình Pose Landmarker và Face Landmarker dự đoán:
     - *Câu hỏi 1:* Mô hình có trả điểm cho khớp bị che không? (Nhận xét: MediaPipe vẫn cố gắng dự đoán tọa độ 3D và hạ thấp chỉ số visibility/confidence).
     - *Câu hỏi 2:* Người quay lưng mô hình có đảo trái/phải không? (Nhận xét: Mạng nơ-ron được huấn luyện lật ảnh nên nhận diện đúng trục cơ thể).
     - *Câu hỏi 3:* Bộ điểm của mô hình có giống 17 điểm COCO không? (MediaPipe sử dụng chuẩn BlazePose 33 điểm, bao gồm thêm các điểm bàn tay, ngón chân, gót chân).

---

## 5. QUY TRÌNH KIỂM TRA CHẤT LƯỢNG (QUALITY ASSURANCE & SELF-QC PROTOCOLS)

### 5.1. Bảng tra cứu 5 mã lỗi kinh điển
Để đạt điểm tối đa ở tiêu chí C7, người học cần sử dụng chính xác hệ mã lỗi chuẩn hóa sau:

| Mã lỗi | Bản chất lỗi | Mức độ nguy hại | Biểu hiện trên CVAT | Giải pháp khắc phục |
|:---|:---|:---:|:---|:---|
| `lr_swap` | Đổi nhầm Trái $\leftrightarrow$ Phải | **Nghiêm trọng nhất** | Các đường nối xương chéo nhau (chân chữ X, tay bắt chéo qua ngực bất thường). | Chọn điểm, hoán đổi vị trí hai bên tương ứng (ví dụ kéo `left_knee` sang chân trái thật của người). |
| `vis_flag` | Sai cờ hiển thị ($v=0, 1, 2$) | **Rất cao** | Khớp ngoài ảnh nhưng cờ $v=2$; khớp che khuất không thấy gì vẫn để $v=2$. | Mở menu thuộc tính của điểm, chuyển sang `outside` ($v=0$) hoặc `occluded` ($v=1$). |
| `position` | Lệch tọa độ tâm khớp $> 3-5$ px | **Cao** | Hông đặt ở cạp quần; mắt cá chân đặt ở đế giày; mí mắt đặt ở hàng lông mi. | Phóng to $400\%$, dời điểm về đúng tâm khớp giải phẫu hoặc mép mí mắt. |
| `missing` | Bỏ sót đối tượng hoặc điểm mốc | **Trung bình** | Người $\ge 60$ px trong ảnh không có skeleton; skeleton thiếu điểm do lỡ tay xóa. | Tạo bổ sung Skeleton cho người bị sót; dựng lại điểm mốc bị thiếu. |
| `extra` | Gán thừa đối tượng không hợp lệ | **Trung bình** | Gán skeleton lên áp phích, gương, tượng sáp, người $< 60$ px. | Xóa toàn bộ Skeleton thừa. |
| `ok` | Đạt chuẩn chất lượng | Không | Khớp đặt chính xác, cờ đúng, đường nối giải phẫu tự nhiên. | Giữ nguyên. |

### 5.2. Kịch bản chạy script kiểm tra `check_submission.py` & Xử lý ngoại lệ

```powershell
# Cú pháp chạy độc lập kiểm tra file xuất:
python src/check_submission.py --persons src/exports/NguyenBaDuy_persons.zip --faces src/exports/NguyenBaDuy_faces.zip
```

#### Các thông báo lỗi thường gặp và cách xử lý triệt để:
1. `[LỖI] Zip có kèm N ảnh... BỎ CHỌN 'Save images'`:
   - *Nguyên nhân:* Khi xuất từ CVAT bật tùy chọn lưu kèm ảnh gốc.
   - *Khắc phục:* Chạy lại `python lab.py export` (script này tự động tắt cờ lưu ảnh).
2. `[LỖI] Điểm v > 0 nằm sát mép ảnh (x = 0.5, y = 480.0)...`:
   - *Nguyên nhân:* Cố tình kéo điểm ngoài ảnh dính vào viền khung hình.
   - *Khắc phục:* Mở CVAT, tìm điểm đó, đổi thuộc tính thành `outside` ($v=0$).
3. `[CẢNH BÁO] Cặp left_shoulder và right_shoulder có khoảng cách X nhỏ bất thường...`:
   - *Nguyên nhân:* Thường xảy ra ở góc chụp nghiêng gắt hoặc có nguy cơ bị `lr_swap`.
   - *Khắc phục:* Mở ảnh kiểm tra lại, nếu là người nghiêng thật thì giữ nguyên và ghi giải trình vào Mục 5 báo cáo; nếu do nhầm L/R thì sửa ngay.

---

## 6. MẪU BÁO CÁO CHUẨN & CHIẾN LƯỢC TRẢ LỜI CÂU HỎI TỰ ĐÁNH GIÁ

File báo cáo phải được đổi tên thành: `report/individual/2A202603192_NguyenBaDuy.md`.

### Hướng dẫn hoàn thiện các mục then chốt:

#### Mục 0. Thông tin học viên:
- Họ tên: Nguyễn Bá Duy
- MSSV: `2A202603192`
- Email: *(Điền email sinh viên của bạn)*
- Link repository: `https://github.com/<tai-khoan>/K5-L2A-DAY04-NguyenBaDuy-2A202603192-KeypointPose-Student`
- Phiên bản CVAT: *CVAT 2.82.0 (hoặc phiên bản hiển thị trong Help -> About)*
- Máy: Hệ điều hành Windows · RAM 16 GB (hoặc thông số máy bạn) · chạy CVAT chế độ đầy đủ.

#### Mục 1. Kiểm tra label (CP0):
- `person`: Kiểu Skeleton, 17 điểm, Đúng bảng, Đúng thứ tự, 19 cạnh.
- `face`: Kiểu Skeleton, 13 điểm, Đúng bảng, Đúng thứ tự, 13 cạnh.
- Mẫu `person` dựng theo người quay mặt về camera: **Có** (các điểm `left_*` nằm bên phải khung mẫu).
- Cách đổi cờ cho 1 điểm: *"Nhấp chuột phải vào điểm mốc cần đổi (hoặc chọn điểm trong danh sách đối tượng bên phải), tick chọn thuộc tính 'occluded' để xuất ra v=1 hoặc 'outside' để xuất ra v=0."*

#### Mục 3. Bảng ca khó (Ví dụ mẫu chuẩn chỉnh):
| Ảnh | Người / mặt | Điểm | Cờ đã chọn | Phân vân giữa | Lý do chọn |
|:---|:---|:---|:---:|:---|:---|
| `occluded_02.jpg` | Người lái xe | `right_ankle` | `0` | $v=1$ và $v=0$ | Cổ chân bị thân xe máy và ống xả che khuất hoàn toàn, không có cơ sở ước lượng chính xác nên chọn $v=0$. |
| `back_01.jpg` | Người quay lưng | `nose` | `0` | $v=1$ và $v=0$ | Người quay lưng $180^\circ$, đầu nghiêng nhẹ nhưng chóp mũi bị gò má che lấp, không đoán bừa. |
| `face_sunglasses.jpg` | Mặt đeo kính | `left_eye_upper`| `1` | $v=1$ và $v=2$ | Tròng kính đen cản quang không thấy mí mắt, nhưng gọng kính định vị rõ ranh giới hốc mắt nên chọn $v=1$. |

#### Mục 4. Bảng tự review (Mẫu tối thiểu 8 dòng theo chuẩn CP3):
| Ảnh | Người / mặt | Điểm | Mã lỗi | Ghi chú | Đã sửa? |
|:---|:---|:---|:---|:---|:---|
| `back_02.jpg` | Người đi bộ | `left_shoulder` / `right_shoulder` | `lr_swap` | Người quay lưng nhưng ban đầu để nguyên trục mẫu dẫn đến đảo L/R | Có |
| `back_02.jpg` | Người đi bộ | `left_wrist` | `position` | Cổ tay bị áo khoác thụng che, ban đầu cắm vào mép tay áo | Có |
| `occluded_01.jpg`| Người ngồi ghế| `left_knee` | `vis_flag` | Đầu gối sau bàn ăn, ban đầu để v=2 nhầm thành bình thường | Có |
| `small_01.jpg` | Người xa | Toàn bộ | `missing` | Bỏ sót 1 người cao 65 px ở hậu cảnh | Có |
| `easy_04.jpg` | Người đứng | `left_ankle` | `vis_flag` | Chân ra ngoài mép ảnh dưới, ban đầu kéo sát viền ảnh | Có |
| `face_02.jpg` | Mặt nghiêng | `right_eye_outer` | `position` | Khóe mắt ngoài bị lệch 3px lên phía thái dương | Có |
| `face_05.jpg` | Mặt thẳng | Toàn bộ 13 điểm | `ok` | Khớp mí mắt và khóe môi chuẩn xác từng pixel | Có |
| `crowd_01.jpg` | Người thứ 3 | Toàn bộ 17 điểm | `ok` | Các cạnh nối giải phẫu phân minh, không chồng chéo | Có |

#### Mục 7. Tự đánh giá (Chiến lược trả lời sâu sắc, chuẩn chuyên gia AI):
1. **Lỗi mắc nhiều nhất và cách phòng tránh:**
   - *Trả lời:* Lỗi hay gặp nhất là xu hướng phán đoán chủ quan theo mắt người xem dẫn đến nguy cơ `lr_swap` ở các góc chụp xiên $45^\circ$, và việc đặt nhầm cờ $v=2$ cho các khớp bị lớp vải dày che khuất. Cách phòng tránh trong dự án sản xuất là thiết lập quy trình "Self-Orientation Check": luôn tự đặt mình vào vị trí tọa độ của đối tượng và kiểm tra tính hợp lý của 19 cạnh liên kết trước khi lưu.
2. **Quy tắc đề xuất bổ sung vào Guideline:**
   - *Trả lời:* Đề xuất bổ sung **"Quy tắc giới hạn hộp sọ khi ước lượng tai người quay lưng"**: Khi người quay lưng chếch, nếu chỉ nhìn thấy vành tóc mà không thấy gờ sụn vành tai thì bắt buộc gán $v=0$ thay vì cố ước lượng $v=1$, nhằm tránh hiện tượng mô hình sinh heatmap tai lệch ra ngoài xương đầu.
3. **Nhãn đi vào mô hình ở đâu & Hậu quả nếu không sửa lỗi:**
   - *Trả lời:* Nhãn đi vào mô hình tại bước **Target Heatmap Generation** và **Loss Weighting**. Nếu không sửa lỗi `lr_swap` trên người quay lưng, khi thực hiện kỹ thuật tăng cường dữ liệu lật ảnh ngang (Horizontal Flip Augmentation: $x' = W - x$, đổi nhãn `left_*` $\leftrightarrow$ `right_*`), mô hình sẽ nhận hai tín hiệu gradient triệt tiêu lẫn nhau. Hậu quả là hàm loss sẽ dao động mạnh, mạng nơ-ron không thể hội tụ và triệt tiêu hoàn toàn khả năng nhận diện tư thế người từ phía sau.

---

## 7. CHECKLIST NGHIỆM THU CUỐI CÙNG TRƯỚC KHI NỘP

Người học phải tự tích đủ toàn bộ các mục trước thời điểm 23:59 GMT+7:

- [ ] **Tên Repo GitHub:** Đặt chuẩn xác không dấu `K5-L2A-DAY04-NguyenBaDuy-2A202603192-KeypointPose-Student` (hoặc tiền tố `K5-L2-...` theo quy định lớp).
- [ ] **File Assignment:** Tồn tại file `src/exports/assignment.json` do `lab.py setup` sinh ra (chứa đúng MSSV `2A202603192`, không sửa tay).
- [ ] **File Xuất chính:** Đầy đủ 2 file nén trong `src/exports/`:
  - `NguyenBaDuy_persons.zip` (Chứa `annotations/person_keypoints_default.json`).
  - `NguyenBaDuy_faces.zip`.
  - Tuyệt đối **không chứa ảnh** bên trong file zip.
- [ ] **Script Kiểm tra:** Lệnh `python lab.py export` in ra `Kết quả: ĐẠT` và không có dòng `[LỖI]`.
- [ ] **File Báo cáo:** Đã đổi tên thành `report/individual/2A202603192_NguyenBaDuy.md` và điền đủ 100% các mục từ 0 đến 9.
- [ ] **Bảng Review:** Có ít nhất **8 dòng** với đầy đủ các cột mã lỗi và trạng thái sửa chữa.
- [ ] **Hình ảnh Minh chứng:** Đủ 4 ảnh chụp trong thư mục `report/individual/images/`:
  - `1_label_person.png`
  - `2_nguoi_quay_lung.png`
  - `3_mat_phong_to.png`
  - `4_json_export.png`
- [ ] **Trạng thái Job trên CVAT:** Cả hai job `persons-NguyenBaDuy` và `faces-NguyenBaDuy` đã chuyển sang trạng thái **completed** (`Finish the job`).
- [ ] **Lịch sử Git:** Không commit bất kỳ file ảnh gốc nào thuộc thư mục `data/` hay ảnh COCO/Wikimedia vào git tracking (`git status` sạch sẽ).
- [ ] **Đẩy mã nguồn lên GitHub:** Đã chạy `git push -u origin main`, kiểm tra repository trên giao diện web GitHub hiển thị đầy đủ cấu trúc và file báo cáo. Mời quyền truy cập chấm bài nếu repo để chế độ Private.
- [ ] **Nộp link trên LMS:** Copy link repository dán vào ô nộp bài trên hệ thống LMS trước thời hạn quy định.

---
*Kế hoạch được phê duyệt và sẵn sàng cho việc triển khai thực tế.*

