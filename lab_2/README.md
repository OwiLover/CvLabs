# Команда: 13  
Участники: Цыгляев Владислав, Габдрахманов Рустам, Лебедев Иван

# Источники

1) https://docs.opencv.org/4.x/d4/dc6/tutorial_py_template_matching.html

2) https://en.wikipedia.org/wiki/Template_matching

3) https://docs.opencv.org/4.x/da/df5/tutorial_py_sift_intro.html

4) https://www.geeksforgeeks.org/sift-interest-point-detector-using-python-opencv/

5) https://en.wikipedia.org/wiki/Scale-invariant_feature_transform

6) https://habr.com/ru/companies/joom/articles/445354/

8) https://docs.opencv.org/4.x/dc/dc3/tutorial_py_matcher.html

# Теория

## template matching

template matching — метод, основанный на нахождении места на изображении, наиболее похожем на шаблон. “Похожесть” изображения задается определенной метрикой. То есть, шаблон "накладывается" на изображение, и считается расхождение между изображением и и шаблоном. Положение шаблона, при котором это расхождение будет минимальным, и будет означать место искомого объекта.


В качестве метрики можно использовать разные варианты, например — сумма квадратов разниц между шаблоном и картинкой (sum of squared differences, SSD), или использовать кросс-корреляцию (cross-correlation, CCORR). Пусть f и g — изображение и шаблон размерами (k, l) и (m, n) соответственно (каналы цвета пока будем игнорировать); i,j — позиция на изображении, к которой мы "приложили" шаблон.

![Alt text](../images/lab_2/readme/math_func_1.png)

Кросс-корреляция на самом деле является сверткой двух изображений. Свёртки можно реализовать быстро, используя быстрое преобразование Фурье. Согласно теореме о свёртке, после преобразования Фурье свёртка превращается в простое поэлементное умножение:

![Alt text](../images/lab_2/readme/math_func_2.png)

В OpenCV уже реализован поиск шаблона с помощью метода matchTemplate (кстати используется именно реализация через FFT), использующий разные метрики расхождений:

- CV_TM_SQDIFF — сумма квадратов разниц значений пикселей
- CV_TM_SQDIFF_NORMED — сумма квадрат разниц цветов, отнормированная в диапазон 0..1.
- CV_TM_CCORR — сумма поэлементных произведений шаблона и сегмента картинки
- CV_TM_CCORR_NORMED — сумма поэлементных произведений, отнормированное в диапазон -1..1.
- CV_TM_CCOEFF — кросс-коррелация изображений без среднего
- CV_TM_CCOEFF_NORMED — кросс-корреляция между изображениями без среднего, отнормированная в -1..1 (корреляция Пирсона)


Можно заметить, что эти метрики требуют попиксельного соответствия шаблона в искомом изображении. Любое отклонение гаммы, света или размера приведут к тому, что методы не будут работать.

Проблему разного цвета и света можно решить применив фильтр нахождения граней (edge detection filter). Этот метод оставляет лишь информацию о том, в каком месте изображения находились резкие перепады цвета.

Также существует проблема разных размеров, однако она уже была решена. Лог-полярная трансформация преобразует картинку в пространство, в котором изменение масштаба и поворот будут проявляться как смещение. Используя эту трансформацию, мы можем восстановить масштаб и угол. После этого, отмасштабировав и повернув шаблон, можно найти и позицию шаблона на исходной картинке. В литературе рассматривается случай, когда по горизонтали и вертикали шаблон изменяется пропорционально, и при этом коэффициент масштаба варьируется в небольших пределах (2.0… 0.8). К сожалению, изменение размеров шаблона может быть бо́льшим и непропорциональным, что может привести к некорректному результату.

## SIFT

Вместо того, чтобы искать шаблон целиком, можно найти его типичные части, например, углы объекта. Кажется, что найти углы проще, так как это мелкие (а значит, простые) объекты. Класс методов нахождения ключевых точек называется “keypoint detection”, а алгоритмы сравнения и поиска картинок с помощью ключевых точек — “keypoint matching”. Поиск шаблона на картинке сводится к применению алгоритма обнаружения ключевых точек к шаблону и картинке, и сопоставлению ключевых точек шаблона и картинки.

Обычно “ключевые точки” находят автоматически, находя пиксели, окружение которых которых обладает определёнными свойствами. Было придумано множество способов и критериев их нахождения. Все эти алгоритмы являются эвристиками, которые находят какие-то характерные элементы изображения, как правило — углы или резкие перепады цвета. Хороший детектор должен работать быстро, и быть устойчивым к трансформациям картинки (при изменении картинки ключевые точки не должны переставать находиться/двигаться).

SIFT (Scale-invariant feature transform) - один из алгоритмов нахождения ключевых точек, который может учитывать изменение масштаба объекта. Возьмем изображение, из которого извлекаем ключевые точки, и начнём постепенно уменьшать его размер с каким-то небольшим шагом, и для каждого варианта масштаба будем находить ключевые точки. Масштабирование — тяжелая процедура, но уменьшение в 2/4/8/… раз можно провести эффективно, пропуская пиксели (в SIFT эти кратные масштабы называются “октавами”). Промежуточные масштабы можно аппроксимировать, применяя к картинке гауссовский блюр с разным размером ядра. Как мы уже описали выше, это можно сделать вычислительно эффективно. Результат будет похож на то, как если бы мы сначала уменьшили картинку, а потом увеличили ее до исходного размера — мелкие детали теряются, изображение становится “замыленным”.

![Alt text](../images/lab_2/readme/sift_img_1.png)

После этой процедуры посчитаем разницу между соседними масштабами. Большие (по модулю) значения в этой разнице получатся, если какая-то мелкая деталь перестает быть видна на следующем уровне масштаба, или, наоборот, следующий уровень масштаба начинает захватывать какую-то деталь, которая на предыдущем не была видна. Этот прием называется DoG, Difference of Gaussian. Можно считать, что большое значение в этой разнице уже является сигналом того, что в этом месте на изображении есть что-то интересное. Но нас интересует тот масштаб, для которого эта ключевая точка будет наиболее выразительной. Для этого будем считать ключевой точкой не только точку, которая отличается от своего окружения, но и отличается сильнее всего среди разных масштабов изображений. Другими словами, выбирать ключевую точку мы будем не только в пространстве X и Y, а в пространстве $ (X, Y, Scale) $. В SIFT это делается путём нахождения точек в DoG (Difference of Gaussians), которые являются локальными максимумами или минимумами в $ 3x3x3 $ кубе пространства $ (X, Y, Scale) $ вокруг неё:

![Alt text](../images/lab_2/readme/sift_img_2.png)

Обычно вышеописанные методы применяют для того, чтобы найти объект из шаблона на снимке, на котором он может быть частично скрыт, быть повернут, или немного искажен. 

Метод хорош, и корректно отрабатывает ситуации, когда искомый объект повернут, его размер изменен, или объект частично скрыт (что хорошо для поиска сложных объектов, или ценника, например). Однако, если на объекте мало точек, за которые можно "зацепиться", или форма объекта меняется слишком сильно, то ключевые точки и их на шаблоне и изображении могут не совпасть. Также, фон с большим количеством мелких деталей может сместить "ключевые точки" или изменить их дескрипторы.

# Результаты работы

## Шаблон, совпадающий с картинкой

- шаблон

![Alt text](../images/lab_2/readme/template_2.png)

- template matching

![Alt text](../images/lab_2/readme/match_2.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_2.png)

## Сильно увеличенный шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_1.png)

- template matching

![Alt text](../images/lab_2/readme/match_1.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_1.png)

## Перевёрнутый шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_3.png)

- template matching

![alt text](../images/lab_2/readme/match_3.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_3.png)

## Шаблон с пустым местом

- шаблон

![Alt text](../images/lab_2/readme/template_4.png)

- template matching

![alt text](../images/lab_2/readme/match_4.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_4.png)

## Шаблон с артефактами

- шаблон

![Alt text](../images/lab_2/readme/template_5.png)

- template matching

![alt text](../images/lab_2/readme/match_5.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_5.png)

## Черно-белый шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_6.png)

- template matching

![alt text](../images/lab_2/readme/match_6.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_6.png)

## Заблюреный шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_7.png)

- template matching

![alt text](../images/lab_2/readme/match_7.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_7.png)

## Сильно уменьшенный шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_8.png)

- template matching

![alt text](../images/lab_2/readme/match_8.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_8.png)

## Шаблон с изменённым ракурсом

- шаблон

![Alt text](../images/lab_2/readme/template_9.png)

- template matching

![alt text](../images/lab_2/readme/match_9.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_9.png)

## Шаблон с изменённым элементом

- шаблон

![Alt text](../images/lab_2/readme/template_10.png)

- template matching

![alt text](../images/lab_2/readme/match_10.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_10.png)

## Сильно изменённый шаблон

- шаблон

![Alt text](../images/lab_2/readme/template_11.png)

- template matching

![alt text](../images/lab_2/readme/match_11.png)

- SIFT

![Alt text](../images/lab_2/readme/sift_11.png)

# Вывод

В результате работы алгоритмов можно сделать вывод, что оба алгоритма (template matching и SIFT) хорошо справляются с определением шаблона, который имеет схожий масштаб и положение с картинкой, на которой производится поиск. Однако если у шаблона изменён размер, он наклонен или повернут относительно картинки, template matching перестаёт справляться с задачей, SIFT в свою очередь всё ещё справляется отлично. SIFT смог правильно определить даже значительно изменённое изображение (изменён цвет, добавлены артефакты, блюр, изменён масштаб, наклон и поворот). Стоит отметить, что единственное слабое место, обнаруженное у SIFT, это сильно заблюренные изображеня, несмотря на то, что алгоритм примерно правильно определил место, рамка вышла сильно хуже, чем у template matching