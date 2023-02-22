# Тестовое задание №2 на позицию Data Scientist (Open Code)

Данный проект посвящен разработке алгоритма автоматического выделения резьбового участка на 3D-модели.

Jupyter-ноутбук с пошаговым описанием алгоритма, а также файлы `.py` с их реализацией находятся в директории `src`. Файл `.stl` с входными данными в можно найти в директории `stl`. 

## Теория

На вход алгоритму подается файл `.stl`, который преобразуется в полигональную сетку $M$ с помощью библиотеки `numpy-stl`. Далее будем работать с сеткой $M$. Ожидается также, что ось вращения ниппеля совпадает с осью $Oy$. Это вполне разумное требование. Однако если алгоритм ничего не знает об ориентации модели, то потребуется реализовать функцию, позволяющую определять ось вращения, а затем повернуть модель соответствующим образом. Последнее осуществляется функциями библиотеки `numpy-stl`. Вычисление оси вращения представлется отдельной содержательной задачей.

Важной особенностью входных данных является строение резьбы: у нее нет наклона (*нельзя закрутить гайку (?), здесь я могу быть не очень точен в терминах*). Данный алгоритм основан на этом свойстве. Для обычной резьбы он не подходит.

### Описание алгоритма

1. Для кажого полигона $p_i$ сетки $M$ задан вектор нормали $\vec{n}_i$ (голубые на рисунке ниже). Первым шагом алгоритм вычисляет углы $\alpha_j$ между векторами $\vec{n}_i$ и вектором $(0,1,0)$, сонаправленным с осью вращения.

<p align="left">
  <img src="https://github.com/inzrv/open-code-test-2/blob/main/pics/pic_1.png" width="400" title="pic_1">
</p>

2. Далее все полигоны с одинаковыми углами $\alpha_j$ объединяются в наборы $P_{\alpha_j}$. Это делается из предположения о том, что полигоны на витке резьбы окажутся в одном множестве $P_{\alpha_j}$. На рисунке ниже полигоны из множеств $P_{\alpha_j}$ покрашены одним цветом.

<p align="left">
  <img src="https://github.com/inzrv/open-code-test-2/blob/main/pics/pic_2.png" width="200" title="pic_2">
</p>

3. Теперь мы должны объединить полигоны из наборов $P_{\alpha_j}$ в компоненты связности $\mathcal{C}^i_{\alpha_j}$. Сделать это можно двумя способами 

    1: Перевести модель в изображение и работать с ним. Данный способ реализован в этом проекте.
    
    2: Продолжать работать с полигонами как с объектами из $\mathbb{R^3}$. Этот подход обсуждается ниже.
  
    Далее мы будем работать с изображением, полученным проецированием полигонов $p_i$ на плоскость $xOy$. При этом вершины $v_i = (x_i, y_i, z_i)$ сетки $M$ отображаются следующим образом: $v_i \to ((x_i, y_i) + (t_x, t_y)) \cdot s$, где $s \in \mathbb{R}$, $(t_x, t_y, t_z)=|\min\limits_i\{ v_i \}|$. 
  
    Число $s \in \mathbb{R}$ требуется для того, чтобы отобразить маленькую (в случае тестовой) модель в картинку большого размера. При увеличении параметра $s$ увеличивается точность алгоритма и растет время его работы. 
  
    Заметим, что данный шаг ограничивает область примнения алгоритма. Если мы считаем, что отрисовка полигонов осуществляется привычным образом (полигоны перекрывают друг друга при проецировании), то резьба внутри детали не будет обнаружена. В данном проекте полигоны отрисовываются без учета их "глубины" по оси $Oz$ и это не сказывается негативно на работе алгоритма, однако данная особенность должна быть учтена.
    
    Для получения компонент связности мы сначала получаем маски $m_{\alpha_j}$, которые выделяют проекции полигонов множества $P_{\alpha_j}$, а затем ищем на данных масках контуры и сохраняем их. На рисунках ниже изображена маска $m_{90^\circ}$ и контуры на ней.
    
  <p float="left">
    <img src="https://github.com/inzrv/open-code-test-2/blob/main/pics/pic_3.png" width="200" />
    <img src="https://github.com/inzrv/open-code-test-2/blob/main/pics/pic_4.png" width="200" /> 
  </p>
  
  4. Для ускорения работы алгоритма можно убрать из рассмотрения маски с небольшим количеством компонент связности. После этого на каждой маске необходимо найти похожие компоненты связности. Если среди компонент связности маски $m_{\alpha_j}$ много похожих на компоненту $\mathcal{C}^i_{\alpha_j} \subset m_{\alpha_j}$, то скорее всего она соответствуют участку резьбы.
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
