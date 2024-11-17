# pl-nko-makaleler-ba-kanl-k_plinko_dosyalar-ekil-html-animate_one_ball-1.gif
plınko/ makaleler/ başkanlık_plinko_dosyaları/ şekil-html /animate_one_ball-1.gif


# Başkanlık Plinko
Matthew Kay 2021-02-01 Kaynak:vignettes/presidential_plinko.Rmd
Aşağıdaki kod, 2020 ABD Başkanlık seçimleri sonucuna ilişkin tahmini bir dağılımın kantil nokta grafiği için bir Plinko panosu oluşturur.

Daha fazla bilgi için presidential-plinko.com ve mjskay/election-galton-board veri havuzunu ziyaret edin .

# Kurmak


```
library(plinko)
library(dplyr)
library(ggplot2)
library(ggdist)
library(ggforce)
library(distributional)

```
 theme_set(theme_ggdist())
## İstenilen hedef 
Diyelim ki FiveThirtyEight'in Biden'ın 2020 ABD Başkanlık seçimlerini kazanma şansına ilişkin olasılık tahminini görüntülemek istiyoruz. Aşağıdaki veri seti, FiveThirtyEight'in 16 Ekim 2020 itibarıyla tahminidir:
   ```
 data(pres_pred_2020, package = "plinko")

pres_pred_2020 %>%
  ggplot(aes(x = total_ev, y = evprob_chal)) +
  geom_col(fill = "gray75") +
  geom_vline(xintercept = 269) +
  coord_cartesian(xlim = c(0, 538)) +
  labs(
    x = "Electoral votes for Biden",
    y = "Predicted probability",
    title = "Predictive distribution for Biden's electoral votes",
    subtitle = "FiveThirtyEight's model of the 2020 US election as of Oct 16, 2020"
  )  

   ```

# Bu belirli veri kümesi zaten bir histogram olarak özetlenmiştir: sütun , yukarıdaki grafikteki evprob_chalher x değeriyle ( ) ilişkilendirilen olasılıktır . Bu özetlenmiş veriler üzerinde ağırlıklı istatistikler (ağırlıklı ortalamalar, kantiller, vb.) kullanarak doğrudan işlem yapabilirken (ben de presidential-plinko.com uygulaması için bunu yaptım ) bu örnek için özetlenmiş verileri önce bir örneğe dönüştüreceğiz. Bu tür veriler daha sık karşılaşılan ve çalışması biraz daha kolay olan verilerdir.total_ev

sample()Verileri (diyelim ki) 5000 kişilik bir örneğe dönüştürmek için olasılıkları ağırlık olarak kullanabilir ve sağlayabiliriz:

set.seed(1234) # for reproducibility
ev_sample = sample(pres_pred_2020$total_ev, 5000, replace = TRUE, prob = pres_pred_2020$evprob_chal)
Verilerin temelde aynı göründüğünü doğrulayabiliriz (örnekleme hatasına kadar):

tibble(ev_sample) %>%
  ggplot(aes(x = ev_sample)) +
  geom_histogram(binwidth = 1, fill = "gray75") +
  geom_vline(xintercept = 269) +
  coord_cartesian(xlim = c(0, 538)) +
  labs(
    x = "Electoral votes for Biden",
    y = "Frequency",
    title = "Predictive distribution for Biden's electoral votes (sampled)",
    subtitle = "FiveThirtyEight's model of the 2020 US election as of Oct 16, 2020"
  )  


# Temel bir tahtanın oluşturulması
Fonksiyon plinko_board()doğrudan sayısal bir vektör üzerinde çağrılabilir; ancak, bu kullanım genellikle ihtiyaç duyduğunuz pano parametrelerini (kutu sayısı, kutu genişliği, vb.) önceden bilmenizi gerektirir. Bu nedenle, genellikle bir olasılık dağılımını temsil eden bir nesne olan dağılımsalplinko_board() bir nesne üzerinde çağrılması daha kolaydır.

# Belirli bir dağılım nesnesi ve istenen bir bölme genişliği ( bin_width) veya bölme sayısı ( n_bin) verildiğinde, plinko_board()o dağılımdan belirli sayıda topu barındırmak için gerekli tahta parametrelerini otomatik olarak hesaplamak için elinden gelenin en iyisini yapacaktır ( n_ball, varsayılan olarak 'dir 50, ancak biz yalnızca bu örnek için kullanacağız 20).

Örnek verilerden bir dağılımsal nesneyi kullanarak oluşturabilir distributional::dist_sample()ve bunu şuraya geçirebiliriz plinko_board():

ev_dist = dist_sample(list(ev_sample))
board = plinko_board(ev_dist, bin_width = 35, n_ball = 20)
board
## A Plinko Board with 20 balls and 14 bins centered at 344.4096
Tahtanın çizilmesi
Kullanarakautoplot()
Plinko tahtası, topların son konumlarına ulaşmak için izleyebilecekleri rastgele yolları otomatik olarak bulacak. Bu yolları, autoplot()bir nesne döndüren fonksiyonla tahtayı çizerek görebiliriz ggplot():

board %>%
  autoplot()
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Varsayılan olarak, çizim Plinko panosu tarafından ima edilen Binom dağılımını kaplar, böylece yaklaşımın kalitesini değerlendirebilirsiniz. Bunu kapatabiliriz:

board %>%
  autoplot(show_dist = FALSE)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Ayrıca, parametresini geçirerek yolları olmayan panoyu da görebilirsiniz show_paths = FALSEve animasyonun belirli karelerini çizebilirsiniz frame:

autoplot(board, show_paths = FALSE, frame = 9)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Elle çizim
Yukarıdakilere benzer bir grafiği, tahtadaki tüm elemanların konumlarını içeren veri çerçevelerini döndüren slot_edges(), pins(), paths(), ve unctions öğelerini kullanarak manuel olarak da oluşturabiliriz:balls()

ggplot() + 
  geom_segment(aes(x = x, y = 0, xend = x, yend = height), data = slot_edges(board), color = "gray75", size = 1) + 
  geom_point(aes(x, y), data = pins(board), shape = 19, color = "#e41a1c", size = 1) +
  geom_path(aes(x = x, y = y, group = ball_id), data = paths(board), alpha = 1/4, size = 1, color = "gray50") +
  geom_circle(aes(x0 = x, y0 = y, r = width/2), data = balls(board), fill = "#1f78b4", color = NA) +
  coord_fixed(expand = FALSE, clip = "off") +
  ylab(NULL) +
  scale_y_continuous(breaks = NULL) +
  theme(
    axis.line.y = element_blank(),
    axis.line.x = element_line(color = "gray75", size = 1)
  ) 


Ancak, çizimi özelleştirmek isterseniz, bu belgede daha sonra açıklanan modify_layer()ve add_layers()işlevlerini kullanmak muhtemelen daha kolaydır; çünkü bunlar, animasyonlu bir çizim oluşturulurken kullanılan kareleri de etkiler.

Bir slot kenarının 269'a düşmesini zorlamak
269Bu panoyu canlandırmaya başlamadan önce, yukarıda otomatik olarak işlenmeyen ek bir kısıtlamayı da ele almak istiyoruz: Toplar aşağıda kalırsa (Trump kazanır) kırmızı, yukarıda kalırsa (Biden kazanır) mavi renklendirmek isteyeceğiz 269. Bunu yapmak için, üzerine düşecek bir yuva kenarına ihtiyacımız var 269.

Mevcut yaklaşımımızın ne kadar iyi performans gösterdiğini görmek için top yollarının gösterimini kapatıp 'e dikey bir çizgi ekleyerek başlayalım 269. , autoplot()bir nesne döndürür ggplot, bu nedenle bu basittir:

autoplot(board, show_paths = FALSE) +
  geom_vline(xintercept = 269)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


269'a en yakın slot kenarı benim zevkime göre biraz fazla uzakta. Seçim oylarında ne kadar yakın olduğunu görebiliriz:

min(abs(board$slot_edges - 269))
## [1] 12.0904
Bu sorunu çözmek için basit bir yaklaşım, memnun olduğumuz bir dizi bölme boyutu (örneğin 35 ila 45 seçim oyu) alıp, yukarıdaki sayıyı en aza indiren bu aralıkta bir bölme genişliği bulmak olabilir:

closest_edge_to_269 = Vectorize(function(bin_width) {
  board = plinko_board(ev_dist, bin_width = bin_width)
  min(abs(board$slot_edges - 269))
})

bin_width = (35:45)[which.min(closest_edge_to_269(35:45))]
bin_width
## [1] 37
37 kutulu tahtanın nasıl göründüğüne bakalım:

board = plinko_board(ev_dist, bin_width = bin_width, n_ball = 20)

autoplot(board, show_paths = FALSE) +
  geom_vline(xintercept = 269)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Bu oldukça iyi görünüyor! Eğer bu konuda gerçekten titiz davranmak istiyorsak , hala 1 seçim oyu kadar bir farkla gerideyiz:

closest_edge_to_269(bin_width)
## [1] 1.4096
Bunu telafi etmek için tahtanın merkezini hafifçe kaydırarak düzeltebiliriz. Bunu yapmak için 269'a en yakın yuva kenarı ile 269 arasındaki işaretli farkı bilmemiz gerekir:

slot_edge_shift = 269 - board$slot_edges[which.min(abs(board$slot_edges - 269))]
slot_edge_shift
## [1] -1.4096
Daha sonra bunu tahtanın merkezini hafifçe kaydırmak için kullanabiliriz

shifted_center = board$center + slot_edge_shift
board = plinko_board(ev_dist, bin_width = bin_width, n_ball = 20, center = shifted_center)

autoplot(board, show_paths = FALSE) +
  geom_vline(xintercept = 269)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Panonun görünümünü özelleştirme
Artık panomuz istediğimiz parametrelere sahip olduğuna göre (hedef dağılımımızla eşleşen bir dağılım ve 269'a düşen bir yuva avantajı), panonun geri kalan görünümünü özelleştirebiliriz.

Limitleri ayarlama
İlk olarak, parametreyi kullanarak panonun sınırlarını ayarlayabiliriz limits. Bu kullanım durumu için olası seçim oylarının tam aralığına kıyasla dağılımı göstermeliyiz:

board = plinko_board(ev_dist, bin_width = bin_width, n_ball = 20, center = shifted_center, limits = c(0, 538))

autoplot(board)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


ggplot katmanlarını özelleştirme
ggplotAyrıca, normalde bir ggplot layer/geom'a geçireceğiniz parametrelerin ardından bir katman adı alan işlevi kullanarak her kare için oluşturulan nesnelerdeki mevcut katmanları özelleştirebiliriz modify_layer(). Sağladığınız estetik eşlemeler, katmandaki mevcut estetikle birleştirilir ve sağladığınız yeni argümanlar mevcut olanları geçersiz kılar.

Örneğin, topun dolgu renginin x konumuna bağlı olmasını ve anahat renginin siyah olmasını istediğinizi varsayalım:

board %>%
  modify_layer("balls", aes(fill = x), color = "black") %>%
  autoplot(show_dist = FALSE)
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


modify_layer()Aşağıdaki katmanları ayarlamak için kullanabilirsiniz :

“slot_edges”: geom_segment()Yuvaların kenarlarını çizen bir A
"pinler": geom_point()Pinleri çizen bir işaret
"yollar": geom_path()Top yollarını çizen bir
"toplar": geom_circle()Topları çeken bir
“dist”: geom_step()Referans binom dağılımını çizen A
autoplot()Tek bir kareyi çizerken, herhangi bir ggplot nesnesinde olduğu gibi, çağrısından sonra ek ggplot nesneleri de ekleyebilirsiniz :

board %>%
  autoplot(show_dist = FALSE, show_paths = FALSE) +
  geom_vline(xintercept = 269, color = "black", alpha = 0.15, size = 1) +
  annotate("label", x = 269, y = 1500, label = "269", hjust = 0.5, color = "gray50")
## Warning in regularize.values(x, y, ties, missing(ties), na.rm = na.rm):
## collapsing to unique 'x' values
## Warning: Computation failed in `stat_dist_slab()`:
## need at least two non-NA values to interpolate


Ancak bu şekilde eklenen katmanlar nesneye kaydedilmez boardve dolayısıyla panoya animasyon uygulandığında görüntülenmez .

boardggplot nesnelerini , tahta canlandırıldığında görüntülenecek şekilde nesneye eklemek için , or add_layers()işlevini çağırmadan önce kullanmalısınız . Aynı örnek şu şekildedir :autoplot()animate()add_layers()

board %>%
  add_layers(
    geom_vline(xintercept = 269, color = "black", alpha = 0.15, size = 1),
    annotate("label", x = 269, y = 1500, label = "269", hjust = 0.5, color = "gray50")
  ) %>%
  autoplot(show_dist = FALSE, show_paths = FALSE, show_target_dist = FALSE)


Statik çizimler olarak ikisi de aynı görünüyor. Ancak, animasyona (sonraki) geçtiğimizde, add_layers()yaklaşımı kullanmamız gerekecek.

Tahtanın canlandırılması
Varsayılan olarak, tahtadaki toplar arasında herhangi bir ara geçiş yapılmaz:

board %>% 
  # width is determined automatically based on height
  animate(fps = 7.5, height = 450)


Büyük ihtimalle top durumları arasında biraz ara geçiş yapmak isteyeceksiniz. Temel kare sayısının 4 katını bir "bounce-out"kolaylaştırma işleviyle (varsayılan) kullanmanın fiziksel olarak daha doğru hissettiren akıcı bir animasyon oluşturduğunu görüyorum. tween_balls()İşlevi kullanarak ara geçiş ekleyebilirsiniz. Ayrıca ek kareleri hesaba katmak için 'ı artırmak isteyeceksiniz fpsve animasyonun sonuna şunu kullanarak 3 saniyelik (90 karelik) bir duraklama ekleyeceğiz end_pause:

board %>%
  tween_balls(frame_mult = 4, ease = "bounce-out") %>%
  animate(fps = 30, height = 450, end_pause = 90)


coresTween nedeniyle artan kare sayısı göz önüne alındığında, argümanını kullanarak kareleri paralel olarak işleyerek hızı artırmak isteyebilirsiniz animate.plinko_board(). coresArgüman varsayılan olarak 'dir getOptions("cores", 1), bu nedenle çekirdek sayısını tüm sonraki çağrılar için kullanılabilir toplam sayıya ayarlamak istiyorsanız animate.plinko_board(), şunları da kullanabilirsiniz:

options(cores = parallel::detectCores())
Test amaçları için (örneğin daha hızlı işleme), son animasyondaki kareleri filtrelemek de yararlı olabilir. Bunu filter_frames(), filtreleme koşullarını aynı biçimde alan filter()ve bunları tarafından döndürülen veri karesine uygulayan kullanarak yapabilirsiniz frames(board). Sütun, yalnızca bir topun düştüğünü göstermek için sütunla ( top bir yuvanın altına çarptıktan ve hareket etmeyi bıraktıktan sonra) "ball_id"birleştirilebilir , örneğin:"stopped"TRUE

board %>%
  filter_frames(ball_id == 1, !stopped) %>%
  tween_balls(frame_mult = 4, ease = "bounce-out") %>%
  animate(fps = 30, height = 450)


Tam, açıklamalı bir pano
Son olarak, mevcut geometrileri ayarlamak için her şeyi bir araya getirebilir modify_layer(), açıklamalar ekleyebilir add_layers(), ara geçiş ekleyebilir tween_balls()ve animasyon uygulayabiliriz:

Biden_color = "#0571b0"
Trump_color = "#ca0020"

annotation_height = 2000

board %>%
  modify_layer("pins", color = "gray50") %>%
  # the "balls" layer uses the frame(board) data frame, which has a "region"
  # column giving the part of the board the ball is in ("start", "pin", or "slot")
  # we can use this to change the ball color when it falls into the slot
  modify_layer(
    "balls", 
    aes(fill = ifelse(region == "slot", ifelse(x <= 269, "Trump", "Biden"), "none")),
    color = NA
  ) %>%
  add_layers(
    geom_vline(xintercept = 269, color = "black", alpha = 0.15, size = 1),
    annotate("text", 
      x = 290, y = 0.92 * annotation_height,
      label = "Biden wins", hjust = 0, color = Biden_color,
    ),
    annotate("text", 
      x = 250, y = 0.92 * annotation_height,
      label = "Trump wins", hjust = 1, color = Trump_color
    ),
    annotate("label", 
      x = 269, y = 0.97 * annotation_height,
      label = "269", hjust = 0.5, color = "gray50",
      fontface = "bold"
    ),
    expand_limits(y = annotation_height),
    scale_fill_manual(
      limits = c("none", "Biden", "Trump"),
      values = c("gray45", Biden_color, Trump_color), 
      guide = FALSE
    ),
    theme(axis.title.x = element_text(hjust = 0, size = 10, color = "gray25")),
    xlab("Electoral votes for Biden")
  ) %>%
  tween_balls(frame_mult = 4, ease = "bounce-out") %>%
  animate(fps = 30, height = 550, end_pause = 90)


Matthew Kay tarafından geliştirildi .

Site pkgdown 1.6.1 ile oluşturuldu.
