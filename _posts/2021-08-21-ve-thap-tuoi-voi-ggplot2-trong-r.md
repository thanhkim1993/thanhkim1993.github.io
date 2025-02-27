---
title: 'Vẽ tháp tuổi với ggplot2 trong R'
date: 2021-08-21
permalink: /posts/2021/08/ve-thap-tuoi-voi-ggplot2-trong-r/
tags:
  - data visualisation
  - applied data science with R
---

Hôm nay mình chia sẻ vẽ tháp tuổi với ggplot2 trong R. Ví dụ được lấy từ nghiên cứu Viêm gan mình đang thực hiện.

Chuẩn bị dữ liệu
======
Mọi người vào trang [GitHub](https://github.com/thanhkim1993/ScienceBlog/tree/main/Plotting-pyramids) của mình để sử dụng dữ liệu và Rscript. Có ba tập tin ở trang này:
* Weighted-Population.csv là dữ liệu từ nghiên cứu CHIME. Dữ liệu này có 3 biến số là nhóm tuổi (age_g5), giới tính (male_u), và số người (population). Biến population ở dạng thập phân vì mình đã weight dữ liệu lại. Chi tiết này không cần phải bận tâm
* HCMC-population.csv là dữ liệu của Thành phố Hồ Chí Minh trong Tổng điều tra dân số 2019. Ba biến số tương tự là Age_group, sex, và population.
* plotting-pyramid.R là Rscript có chứa code. Code mình viết theo dạng tái lặp (reproducible) nên mọi người có thể tải về rồi chạy luôn.

OK, chúng ta bắt đầu.

Vẽ tháp tuổi của CHIME
======
### Bước 1: tải các package cần
```r
library(ggplot2) # Vẽ hình
library(patchwork) # Kết nối các hình với nhau
```
### Bước 2: tải dữ liệu vào R

Các bạn có thể vào trang [GitHub](https://github.com/thanhkim1993/ScienceBlog/tree/main/Plotting-pyramids) để tải về máy, rồi dùng chức năng _**Import Dataset**_ để import tập tin .csv vào R. Ở đây mình giới thiệu cách sử dụng lệnh để tải trực tiếp từ trang GitHub. Việc này giúp ta thao tác nhanh hơn, và có tính tái lặp cao hơn. Lưu ý cần có Internet.

```r
linkURL <-"https://raw.githubusercontent.com/thanhkim1993/ScienceBlog/main/Plotting-pyramids/Weighted-Population.csv"  # Lưu đường link
temporary <- tempfile(pattern = "Weighted-Population", fileext = ".csv") # Tạo một đường dẫn tạm thời trong máy tính
download.file(linkURL, temporary, method = "curl") # Tải tập tin csv về máy theo đường dẫn tạm thời
population <- read.csv(temporary) # Import tập tin csv vào R
unlink(temporary) # Xóa tập tin csv và đường dẫn tương ứng. Đây là bước cần thiết để tránh ứ đọng file không cần thiết
```

### Bước 3: khai báo data set và trục tọa độ cho ggplot2
Lúc này chỉ thấy trục tọa độ, chưa có hình.

```r
ggplot(population, aes(x = age_g5, y = population, fill = male_u))  # lệnh `fill = male_u` nhằm phân tầng theo giới tính
```

![image](https://user-images.githubusercontent.com/62500890/130346579-f84c64b5-852c-47b2-971f-e52f49808b7d.png)

### Bước 4: thêm biểu đồ cột
```r
ggplot(population, aes(x = age_g5, y = population, fill = male_u)) +  # Lưu ý ở đây có dấu cộng. Xuống hàng cho nó gọn.
  geom_bar(stat = "identity") # `geom_bar(stat = "identity")` chức năng y hệt `geom_bar()`
```

![image](https://user-images.githubusercontent.com/62500890/130346617-f300c37e-336a-4caf-8031-6210620ef024.png)

### Bước 5: vẽ biểu đồ nằm ngang + tách biệt nam và nữ bằng trục giữa cho giống tháp tuổi

```r
ggplot(population, aes(x = age_g5, y = ifelse(male_u == "Male", -population,population), fill = male_u)) +
  geom_bar(stat = "identity") 
```

![image](https://user-images.githubusercontent.com/62500890/130346641-7f464316-6d7f-4128-a6d7-40f7889f0ac2.png)

### Bước 6: vẽ biểu đồ ở phương thẳng đứng
```r
ggplot(population, aes(x = age_g5, y = ifelse(male_u == "Male", -population,population), fill = male_u)) +
  geom_bar(stat = "identity") +
  coord_flip() # Lệnh này giúp xoay hình sang phương thẳng đứng
```

![image](https://user-images.githubusercontent.com/62500890/130346682-0c180a5b-e2b5-475f-b9d6-80f8da2dbfa6.png)

### Bước 7: format thông tin trên trục x, y, chỉnh lại màu sắc, và gán vào `chart1`
```r
chart1 <- ggplot(population, aes(x = age_g5, y = ifelse(male_u == "Male", -population/1000,population/1000), fill = male_u)) + 
  geom_bar(stat = "identity") +
  coord_flip() +
  scale_fill_manual(name = "Sex",   # Đặt tên legend
                    breaks = c("Female", "Male"),
                    values = c("blue","red")) + #Chỉnh màu
  scale_y_continuous(labels = abs, # trị truyệt đối cột y
                     limits = max(population$population/1000)*c(-1,1)) + # Chỉnh giới hạn cột y cho 2 bên đều nhau
  labs(title = "Weighted popoulation pyramid in CHIME", x = "Age group", y = "Population (thousand)") +
  theme_bw() # Chỉnh theme màu này cho sáng hình
 
print(chart1)
```

![image](https://user-images.githubusercontent.com/62500890/130346706-f441e706-12a5-4650-98a0-67ac1f82c102.png)

Vẽ tháp tuổi của Thành phố Hồ Chí Minh
=====
Tương tự như vậy, ta vẽ tháp tuổi của Thành phố Hồ CHí Minh trong năm 2019. Tiếp tục tải dữ liệu TPHCM về từ GitHub.

```r
linkURL <- "https://raw.githubusercontent.com/thanhkim1993/ScienceBlog/main/Plotting-pyramids/HCMC-population.csv"
temporary <- tempfile(pattern = "HCMC-Population", fileext = ".csv")
download.file(linkURL, temporary, method = "curl")
HCMCpopulation <- read.csv(temporary)
unlink(temporary)
```
Vẽ tháp tuổi TPHCM, và gán vào `chart2`.
```r
chart2 <- ggplot(HCMCpopulation, 
                         aes(x=Age_group, 
                             y = ifelse(sex == "Female", population/1000, -population/1000), 
                             fill = sex)) +
  geom_col() + coord_flip() +
  scale_y_continuous(labels = abs, limits = max(HCMCpopulation$population)*c(-0.001,0.001)) +
  scale_fill_manual(name="Sex",
                    breaks=c("Female", "Male"),
                    values = c("blue","red")) + labs(x = "Age group", y = "Population (thousand)")  +
  labs(title = "Population pyramid in Ho Chi MInh City")  + theme_bw()
 
print(chart2)
```

![image](https://user-images.githubusercontent.com/62500890/130346772-44c8a8b5-765a-4fe8-a888-784364c88326.png)

Gộp hai tháp tuổi
=====
Cuối cùng, chúng ta gộp hai tháp tuổi với nhau để dễ so sánh. Lúc này package `patchwork` phát huy tác dụng của nó.

```r
appended.chart <- chart1 + chart2
print(appended.chart)
```

![image](https://user-images.githubusercontent.com/62500890/130346819-63aabb54-1d24-4b58-a759-1b9e2c3bd14f.png)

Kết luận
=====
Tới đây, ta có thể dễ dàng so sánh phân bố tuổi và giới tính trong nghiên cứu CHIME với số liệu của thành phố. Chúng ta có thể phần nào kết luận nghiên cứu CHIME tập trung vào những người lớn tuổi – nhóm người có tỉ lệ hiện mắc viêm gan siêu vi C cao hơn các nhóm tuổi khác. Điều này có thể khiến con số ước lượng tỉ lệ nhiễm viêm gan siêu vi C trong nghiên cứu cao hơn con số thực tế của thành phố. Hiện tượng được gọi là Ước lượng trội, hoặc Overestimate.
