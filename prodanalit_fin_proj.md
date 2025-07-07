```python
import numpy as np
import pandas as pd
import scipy.stats as ss
from operator import attrgetter
import seaborn as sns
import plotly.express as px
import matplotlib.pyplot as plt
from matplotlib import colors as mcolors
```


```python
# Загрузка данных
customers   = pd.read_csv('olist_customers_dataset.csv') # таблица с уникальными идентификаторами пользователей
orders      = pd.read_csv('olist_orders_dataset.csv') # таблица заказов
order_items = pd.read_csv('olist_order_items_dataset.csv') # товарные позиции
```


```python
customers.info()
customers.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 99441 entries, 0 to 99440
    Data columns (total 5 columns):
     #   Column                    Non-Null Count  Dtype 
    ---  ------                    --------------  ----- 
     0   customer_id               99441 non-null  object
     1   customer_unique_id        99441 non-null  object
     2   customer_zip_code_prefix  99441 non-null  int64 
     3   customer_city             99441 non-null  object
     4   customer_state            99441 non-null  object
    dtypes: int64(1), object(4)
    memory usage: 3.8+ MB





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customer_id</th>
      <th>customer_unique_id</th>
      <th>customer_zip_code_prefix</th>
      <th>customer_city</th>
      <th>customer_state</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>06b8999e2fba1a1fbc88172c00ba8bc7</td>
      <td>861eff4711a542e4b93843c6dd7febb0</td>
      <td>14409</td>
      <td>franca</td>
      <td>SP</td>
    </tr>
    <tr>
      <th>1</th>
      <td>18955e83d337fd6b2def6b18a428ac77</td>
      <td>290c77bc529b7ac935b93aa66c333dc3</td>
      <td>9790</td>
      <td>sao bernardo do campo</td>
      <td>SP</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4e7b3e00288586ebd08712fdd0374a03</td>
      <td>060e732b5b29e8181a18229c7b0b2b5e</td>
      <td>1151</td>
      <td>sao paulo</td>
      <td>SP</td>
    </tr>
    <tr>
      <th>3</th>
      <td>b2b6027bc5c5109e529d4dc6358b12c3</td>
      <td>259dac757896d24d7702b9acbbff3f3c</td>
      <td>8775</td>
      <td>mogi das cruzes</td>
      <td>SP</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4f2d8ab171c80ec8364f7c12e35b23ad</td>
      <td>345ecd01c38d18a9036ed96c73b8d066</td>
      <td>13056</td>
      <td>campinas</td>
      <td>SP</td>
    </tr>
  </tbody>
</table>
</div>



Таблица olist_customers_dataset.csv содержит данные о клиентах.

customer_id — позаказный идентификатор пользователя

customer_unique_id — уникальный идентификатор пользователя (аналог номера паспорта)

customer_zip_code_prefix — почтовый индекс пользователя

customer_city — город доставки пользователя

customer_state — штат доставки пользователя


```python
orders.info()
orders.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 99441 entries, 0 to 99440
    Data columns (total 8 columns):
     #   Column                         Non-Null Count  Dtype 
    ---  ------                         --------------  ----- 
     0   order_id                       99441 non-null  object
     1   customer_id                    99441 non-null  object
     2   order_status                   99441 non-null  object
     3   order_purchase_timestamp       99441 non-null  object
     4   order_approved_at              99281 non-null  object
     5   order_delivered_carrier_date   97658 non-null  object
     6   order_delivered_customer_date  96476 non-null  object
     7   order_estimated_delivery_date  99441 non-null  object
    dtypes: object(8)
    memory usage: 6.1+ MB





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>customer_id</th>
      <th>order_status</th>
      <th>order_purchase_timestamp</th>
      <th>order_approved_at</th>
      <th>order_delivered_carrier_date</th>
      <th>order_delivered_customer_date</th>
      <th>order_estimated_delivery_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>e481f51cbdc54678b7cc49136f2d6af7</td>
      <td>9ef432eb6251297304e76186b10a928d</td>
      <td>delivered</td>
      <td>2017-10-02 10:56:33</td>
      <td>2017-10-02 11:07:15</td>
      <td>2017-10-04 19:55:00</td>
      <td>2017-10-10 21:25:13</td>
      <td>2017-10-18 00:00:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>53cdb2fc8bc7dce0b6741e2150273451</td>
      <td>b0830fb4747a6c6d20dea0b8c802d7ef</td>
      <td>delivered</td>
      <td>2018-07-24 20:41:37</td>
      <td>2018-07-26 03:24:27</td>
      <td>2018-07-26 14:31:00</td>
      <td>2018-08-07 15:27:45</td>
      <td>2018-08-13 00:00:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>47770eb9100c2d0c44946d9cf07ec65d</td>
      <td>41ce2a54c0b03bf3443c3d931a367089</td>
      <td>delivered</td>
      <td>2018-08-08 08:38:49</td>
      <td>2018-08-08 08:55:23</td>
      <td>2018-08-08 13:50:00</td>
      <td>2018-08-17 18:06:29</td>
      <td>2018-09-04 00:00:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>949d5b44dbf5de918fe9c16f97b45f8a</td>
      <td>f88197465ea7920adcdbec7375364d82</td>
      <td>delivered</td>
      <td>2017-11-18 19:28:06</td>
      <td>2017-11-18 19:45:59</td>
      <td>2017-11-22 13:39:59</td>
      <td>2017-12-02 00:28:42</td>
      <td>2017-12-15 00:00:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ad21c59c0840e6cb83a9ceb5573f8159</td>
      <td>8ab97904e6daea8866dbdbc4fb7aad2c</td>
      <td>delivered</td>
      <td>2018-02-13 21:18:39</td>
      <td>2018-02-13 22:20:29</td>
      <td>2018-02-14 19:46:34</td>
      <td>2018-02-16 18:17:02</td>
      <td>2018-02-26 00:00:00</td>
    </tr>
  </tbody>
</table>
</div>



order_id — уникальный идентификатор заказа (номер чека)

customer_id — позаказный идентификатор пользователя

order_status — статус заказа

order_purchase_timestamp — время создания заказа

order_approved_at — время подтверждения оплаты заказа

order_delivered_carrier_date — время передачи заказа в логистическую службу

order_delivered_customer_date — время доставки заказа

order_estimated_delivery_date — обещанная дата доставки

Уникальные статусы заказов в таблице olist_orders_dataset:

created — создан;

approved — подтверждён;

invoiced — выставлен счёт;

processing — в процессе сборки заказа;

shipped — отгружён со склада;

delivered — доставлен пользователю;

unavailable — недоступен;

canceled — отменён.


```python
order_items.info()
order_items.head()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 112650 entries, 0 to 112649
    Data columns (total 7 columns):
     #   Column               Non-Null Count   Dtype  
    ---  ------               --------------   -----  
     0   order_id             112650 non-null  object 
     1   order_item_id        112650 non-null  int64  
     2   product_id           112650 non-null  object 
     3   seller_id            112650 non-null  object 
     4   shipping_limit_date  112650 non-null  object 
     5   price                112650 non-null  float64
     6   freight_value        112650 non-null  float64
    dtypes: float64(2), int64(1), object(4)
    memory usage: 6.0+ MB





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_id</th>
      <th>order_item_id</th>
      <th>product_id</th>
      <th>seller_id</th>
      <th>shipping_limit_date</th>
      <th>price</th>
      <th>freight_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00010242fe8c5a6d1ba2dd792cb16214</td>
      <td>1</td>
      <td>4244733e06e7ecb4970a6e2683c13e61</td>
      <td>48436dade18ac8b2bce089ec2a041202</td>
      <td>2017-09-19 09:45:35</td>
      <td>58.90</td>
      <td>13.29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00018f77f2f0320c557190d7a144bdd3</td>
      <td>1</td>
      <td>e5f2d52b802189ee658865ca93d83a8f</td>
      <td>dd7ddc04e1b6c2c614352b383efe2d36</td>
      <td>2017-05-03 11:05:13</td>
      <td>239.90</td>
      <td>19.93</td>
    </tr>
    <tr>
      <th>2</th>
      <td>000229ec398224ef6ca0657da4fc703e</td>
      <td>1</td>
      <td>c777355d18b72b67abbeef9df44fd0fd</td>
      <td>5b51032eddd242adc84c38acab88f23d</td>
      <td>2018-01-18 14:48:30</td>
      <td>199.00</td>
      <td>17.87</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00024acbcdf0a6daa1e931b038114c75</td>
      <td>1</td>
      <td>7634da152a4610f1595efa32f14722fc</td>
      <td>9d7a1d34a5052409006425275ba1c2b4</td>
      <td>2018-08-15 10:10:18</td>
      <td>12.99</td>
      <td>12.79</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00042b26cf59d7ce69dfabb4e55b4fd9</td>
      <td>1</td>
      <td>ac6c3623068f30de03045865e4e10089</td>
      <td>df560393f3a51e74553ab94004ba5c87</td>
      <td>2017-02-13 13:57:51</td>
      <td>199.90</td>
      <td>18.14</td>
    </tr>
  </tbody>
</table>
</div>



order_items  — товарные позиции, входящие в заказы

order_id — уникальный идентификатор заказа (номер чека)

order_item_id — идентификатор товара внутри одного заказа

product_id — ид товара (аналог штрихкода)

seller_id — ид производителя товара

shipping_limit_date — максимальная дата доставки продавцом для передачи заказа партнеру по логистике

price — цена за единицу товара

freight_value — вес товара 

Пример структуры данных можно визуализировать по order_id == 00143d0f86d6fbd9f9b38ab440ac16f5

# Задачи
Вы поразмышляли и сформулировали список задач:

Задача 1: Оценить месячный retention в оформление заказа с помощью когортного анализа.

Задача 2: Определить, существует ли product/market fit у этого маркетплейса.

Задача 3: Определить 5 основных метрик, на которых продакту можно сконцентрироваться, чтобы максимизировать прибыль компании.

Задача 4: Выбрать одну из 3 основных гипотез с помощью фреймворка ICE.

Задача 5: Сформулировать нужные метрики, на которые ваша гипотеза должна повлиять.

Задача 6: Сформулировать выводы о проделанной работе. 

# Задача 1.

Оценить месячный retention в оформление заказа с помощью когортного анализа.

На первом этапе вы решили посмотреть на метрики маркетплейса и на возвращаемость клиента в продукт.

Для этого вам нужно:

Оценить месячный retention в оформление заказа с помощью когортного анализа, так как важно, чтобы клиенты возвращались в маркетплейс для совершения больших покупок.

В рамках исследования необходимо:

Исследовать датасет и определить, какой вид заказа будет учитываться в retention
Построить месячный retention
Проанализировать, чему равен медианный retention 1-го месяца
Найти когорту с самым высоким retention на 3-й месяц.


```python
# Объединение данных в единый датасет data_marketplace
data_marketplace = pd.merge(customers, orders, on='customer_id', how='inner')
data_marketplace = pd.merge(data_marketplace, order_items, on='order_id', how='inner')
```


```python
# Преобразование временных меток в формат datetime
data_marketplace['order_purchase_timestamp'] = pd.to_datetime(data_marketplace['order_purchase_timestamp'])
data_marketplace['order_delivered_customer_date'] = pd.to_datetime(data_marketplace['order_delivered_customer_date'], errors='coerce')
```


```python
# Создаем копию данных, чтобы избежать изменений в исходном DataFrame
data_copy = data_marketplace.copy()
```


```python
# Анализируем только доставленные заказы, чтобы избежать искажений из-за отмен и логистических сбоев.
data_copy = data_copy[data_copy['order_status'] == 'delivered']
```


```python
# Извлекаем месяц и день заказа
data_copy['order_period'] = data_copy['order_purchase_timestamp'].dt.to_period('M')
data_copy['order_period_day'] = data_copy['order_purchase_timestamp'].dt.to_period('D')
```


```python
# Присваиваем когорты на основе даты первого действия для каждого пользователя
data_copy['cohort'] = data_copy.groupby('customer_unique_id')['order_purchase_timestamp'].transform('min').dt.to_period('M')
data_copy['cohort_day'] = data_copy.groupby('customer_unique_id')['order_purchase_timestamp'].transform('min').dt.to_period('D')
```


```python
# Рассчитываем номер периода в днях (месячные периоды, деля на 30)
data_copy['period_number_month'] = np.floor((data_copy.order_period_day - data_copy.cohort_day).apply(attrgetter('n')) / 30)
```


```python
# Агрегируем данные по когортам и рассчитанному номеру периода (в месяцах)
df_cohort = data_copy.groupby(['cohort', 'period_number_month']).agg(n_customers=('customer_unique_id', 'nunique')).reset_index()
```


```python
# Создаем сводную таблицу для когортного анализа
cohort_pivot = df_cohort.pivot_table(index='cohort', columns='period_number_month', values='n_customers')
cohort_pivot
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>period_number_month</th>
      <th>0.0</th>
      <th>1.0</th>
      <th>2.0</th>
      <th>3.0</th>
      <th>4.0</th>
      <th>5.0</th>
      <th>6.0</th>
      <th>7.0</th>
      <th>8.0</th>
      <th>9.0</th>
      <th>...</th>
      <th>11.0</th>
      <th>12.0</th>
      <th>13.0</th>
      <th>14.0</th>
      <th>15.0</th>
      <th>16.0</th>
      <th>17.0</th>
      <th>19.0</th>
      <th>20.0</th>
      <th>21.0</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-09</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10</th>
      <td>262.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2016-12</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01</th>
      <td>717.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02</th>
      <td>1628.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>6.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03</th>
      <td>2503.0</td>
      <td>7.0</td>
      <td>13.0</td>
      <td>11.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04</th>
      <td>2256.0</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>1.0</td>
      <td>6.0</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>10.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05</th>
      <td>3451.0</td>
      <td>14.0</td>
      <td>14.0</td>
      <td>8.0</td>
      <td>13.0</td>
      <td>11.0</td>
      <td>11.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>11.0</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-06</th>
      <td>3037.0</td>
      <td>13.0</td>
      <td>15.0</td>
      <td>11.0</td>
      <td>5.0</td>
      <td>16.0</td>
      <td>9.0</td>
      <td>6.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>9.0</td>
      <td>3.0</td>
      <td>9.0</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-07</th>
      <td>3752.0</td>
      <td>14.0</td>
      <td>10.0</td>
      <td>9.0</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>10.0</td>
      <td>7.0</td>
      <td>5.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>9.0</td>
      <td>6.0</td>
      <td>7.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-08</th>
      <td>4057.0</td>
      <td>27.0</td>
      <td>10.0</td>
      <td>14.0</td>
      <td>15.0</td>
      <td>19.0</td>
      <td>10.0</td>
      <td>10.0</td>
      <td>6.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>7.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-09</th>
      <td>4004.0</td>
      <td>21.0</td>
      <td>18.0</td>
      <td>13.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>11.0</td>
      <td>6.0</td>
      <td>13.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-10</th>
      <td>4328.0</td>
      <td>26.0</td>
      <td>4.0</td>
      <td>5.0</td>
      <td>13.0</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>17.0</td>
      <td>11.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-11</th>
      <td>7060.0</td>
      <td>25.0</td>
      <td>19.0</td>
      <td>11.0</td>
      <td>11.0</td>
      <td>14.0</td>
      <td>8.0</td>
      <td>13.0</td>
      <td>5.0</td>
      <td>2.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-12</th>
      <td>5338.0</td>
      <td>14.0</td>
      <td>19.0</td>
      <td>10.0</td>
      <td>16.0</td>
      <td>9.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>5.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-01</th>
      <td>6842.0</td>
      <td>22.0</td>
      <td>22.0</td>
      <td>23.0</td>
      <td>11.0</td>
      <td>13.0</td>
      <td>17.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-02</th>
      <td>6288.0</td>
      <td>22.0</td>
      <td>23.0</td>
      <td>17.0</td>
      <td>15.0</td>
      <td>12.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-03</th>
      <td>6774.0</td>
      <td>25.0</td>
      <td>10.0</td>
      <td>16.0</td>
      <td>11.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-04</th>
      <td>6582.0</td>
      <td>30.0</td>
      <td>11.0</td>
      <td>15.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-05</th>
      <td>6506.0</td>
      <td>19.0</td>
      <td>21.0</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-06</th>
      <td>5878.0</td>
      <td>22.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-07</th>
      <td>5949.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-08</th>
      <td>6144.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>23 rows × 21 columns</p>
</div>




```python
# Рассчитываем размеры когорт (первый столбец сводной таблицы)
cohort_size = cohort_pivot.iloc[:, 0]
```


```python
# Вычисляем коэффициенты удержания, деля на размер когорты
retention_matrix = cohort_pivot.divide(cohort_size, axis=0)
retention_matrix
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>period_number_month</th>
      <th>0.0</th>
      <th>1.0</th>
      <th>2.0</th>
      <th>3.0</th>
      <th>4.0</th>
      <th>5.0</th>
      <th>6.0</th>
      <th>7.0</th>
      <th>8.0</th>
      <th>9.0</th>
      <th>...</th>
      <th>11.0</th>
      <th>12.0</th>
      <th>13.0</th>
      <th>14.0</th>
      <th>15.0</th>
      <th>16.0</th>
      <th>17.0</th>
      <th>19.0</th>
      <th>20.0</th>
      <th>21.0</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-09</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>...</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>0.007634</td>
      <td>0.003817</td>
      <td>0.003817</td>
    </tr>
    <tr>
      <th>2016-12</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01</th>
      <td>1.0</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>0.005579</td>
      <td>0.001395</td>
      <td>0.001395</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>0.005579</td>
      <td>0.004184</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>0.001395</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02</th>
      <td>1.0</td>
      <td>0.001229</td>
      <td>0.001843</td>
      <td>0.003686</td>
      <td>0.002457</td>
      <td>0.001229</td>
      <td>0.002457</td>
      <td>0.000614</td>
      <td>0.001229</td>
      <td>0.002457</td>
      <td>...</td>
      <td>0.003686</td>
      <td>0.000614</td>
      <td>0.001843</td>
      <td>0.001229</td>
      <td>0.000614</td>
      <td>0.000614</td>
      <td>0.001843</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03</th>
      <td>1.0</td>
      <td>0.002797</td>
      <td>0.005194</td>
      <td>0.004395</td>
      <td>0.001199</td>
      <td>0.000799</td>
      <td>0.003596</td>
      <td>0.001598</td>
      <td>0.002397</td>
      <td>0.002397</td>
      <td>...</td>
      <td>0.001598</td>
      <td>0.001598</td>
      <td>0.001199</td>
      <td>0.002797</td>
      <td>0.000799</td>
      <td>0.001199</td>
      <td>0.000799</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04</th>
      <td>1.0</td>
      <td>0.003546</td>
      <td>0.002660</td>
      <td>0.000443</td>
      <td>0.002660</td>
      <td>0.003546</td>
      <td>0.002660</td>
      <td>0.004433</td>
      <td>0.002216</td>
      <td>0.002660</td>
      <td>...</td>
      <td>0.001330</td>
      <td>NaN</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05</th>
      <td>1.0</td>
      <td>0.004057</td>
      <td>0.004057</td>
      <td>0.002318</td>
      <td>0.003767</td>
      <td>0.003187</td>
      <td>0.003187</td>
      <td>0.001159</td>
      <td>0.002898</td>
      <td>0.002608</td>
      <td>...</td>
      <td>0.003187</td>
      <td>0.001739</td>
      <td>0.000869</td>
      <td>0.002028</td>
      <td>0.001159</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-06</th>
      <td>1.0</td>
      <td>0.004281</td>
      <td>0.004939</td>
      <td>0.003622</td>
      <td>0.001646</td>
      <td>0.005268</td>
      <td>0.002963</td>
      <td>0.001976</td>
      <td>0.001317</td>
      <td>0.003293</td>
      <td>...</td>
      <td>0.002963</td>
      <td>0.000988</td>
      <td>0.002963</td>
      <td>0.001317</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-07</th>
      <td>1.0</td>
      <td>0.003731</td>
      <td>0.002665</td>
      <td>0.002399</td>
      <td>0.003198</td>
      <td>0.001866</td>
      <td>0.002665</td>
      <td>0.001866</td>
      <td>0.001333</td>
      <td>0.002665</td>
      <td>...</td>
      <td>0.002399</td>
      <td>0.001599</td>
      <td>0.001866</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-08</th>
      <td>1.0</td>
      <td>0.006655</td>
      <td>0.002465</td>
      <td>0.003451</td>
      <td>0.003697</td>
      <td>0.004683</td>
      <td>0.002465</td>
      <td>0.002465</td>
      <td>0.001479</td>
      <td>0.001725</td>
      <td>...</td>
      <td>0.001725</td>
      <td>0.000739</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-09</th>
      <td>1.0</td>
      <td>0.005245</td>
      <td>0.004496</td>
      <td>0.003247</td>
      <td>0.003497</td>
      <td>0.002248</td>
      <td>0.002747</td>
      <td>0.001499</td>
      <td>0.003247</td>
      <td>0.001748</td>
      <td>...</td>
      <td>0.000250</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-10</th>
      <td>1.0</td>
      <td>0.006007</td>
      <td>0.000924</td>
      <td>0.001155</td>
      <td>0.003004</td>
      <td>0.001386</td>
      <td>0.002773</td>
      <td>0.003928</td>
      <td>0.002542</td>
      <td>0.002079</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-11</th>
      <td>1.0</td>
      <td>0.003541</td>
      <td>0.002691</td>
      <td>0.001558</td>
      <td>0.001558</td>
      <td>0.001983</td>
      <td>0.001133</td>
      <td>0.001841</td>
      <td>0.000708</td>
      <td>0.000283</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-12</th>
      <td>1.0</td>
      <td>0.002623</td>
      <td>0.003559</td>
      <td>0.001873</td>
      <td>0.002997</td>
      <td>0.001686</td>
      <td>0.000937</td>
      <td>0.001124</td>
      <td>0.000937</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-01</th>
      <td>1.0</td>
      <td>0.003215</td>
      <td>0.003215</td>
      <td>0.003362</td>
      <td>0.001608</td>
      <td>0.001900</td>
      <td>0.002485</td>
      <td>0.000877</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-02</th>
      <td>1.0</td>
      <td>0.003499</td>
      <td>0.003658</td>
      <td>0.002704</td>
      <td>0.002385</td>
      <td>0.001908</td>
      <td>0.000954</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-03</th>
      <td>1.0</td>
      <td>0.003691</td>
      <td>0.001476</td>
      <td>0.002362</td>
      <td>0.001624</td>
      <td>0.000443</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-04</th>
      <td>1.0</td>
      <td>0.004558</td>
      <td>0.001671</td>
      <td>0.002279</td>
      <td>0.000912</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-05</th>
      <td>1.0</td>
      <td>0.002920</td>
      <td>0.003228</td>
      <td>0.000615</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-06</th>
      <td>1.0</td>
      <td>0.003743</td>
      <td>0.001021</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-07</th>
      <td>1.0</td>
      <td>0.001009</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-08</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>23 rows × 21 columns</p>
</div>




```python
# расчитываем медианный retention 1-го месяца
median_retention = retention_matrix[1.0].median()
median_retention
```




    0.0035460992907801418




```python
# ищем когорту с самым высоким retention на 3-й месяц.
retention_matrix[3.0]
```




    cohort
    2016-09         NaN
    2016-10         NaN
    2016-12         NaN
    2017-01    0.005579
    2017-02    0.003686
    2017-03    0.004395
    2017-04    0.000443
    2017-05    0.002318
    2017-06    0.003622
    2017-07    0.002399
    2017-08    0.003451
    2017-09    0.003247
    2017-10    0.001155
    2017-11    0.001558
    2017-12    0.001873
    2018-01    0.003362
    2018-02    0.002704
    2018-03    0.002362
    2018-04    0.002279
    2018-05    0.000615
    2018-06         NaN
    2018-07         NaN
    2018-08         NaN
    Freq: M, Name: 3.0, dtype: float64




```python
retention_matrix[3.0].idxmax()
```




    Period('2017-01', 'M')




```python
# Строим тепловую карту когортного анализа
with sns.axes_style("white"):
    fig, ax = plt.subplots(1, 2, figsize=(16, 12), sharey=True, gridspec_kw={'width_ratios': [1, 11]})

    # Тепловая карта для коэффициентов удержания
    sns.heatmap(retention_matrix,
                mask=retention_matrix.isnull(),
                annot=True,
                fmt='.0%',
                cmap='RdYlGn',
                ax=ax[1])
    ax[1].set_title('Retention', fontsize=16)
    ax[1].set(xlabel='№ периода', ylabel='Когорта')

    # Тепловая карта для размеров когорт
    cohort_size_df = pd.DataFrame(cohort_size).rename(columns={0: 'cohort_size'})
    white_cmap = mcolors.ListedColormap(['white'])
    sns.heatmap(cohort_size_df,
                annot=True,
                cbar=False,
                fmt='g',
                cmap=white_cmap,
                ax=ax[0])

    fig.tight_layout()
    plt.show()
```


![png](output_24_0.png)



```python

```

# Задача 2. 

Определить, существует ли product/market fit у маркетплейса.

Построив retention, вы решили оценить, насколько хорошо продукт закрывает потребности клиента.

Для этого вам нужно:

Определить, существует ли product/market fit у этого маркетплейса. Ведь до сих пор непонятно, можно ли масштабировать подобный продукт на новые рынки. Есть вероятность, что маркетплейс будет приносить убытки.

В рамках исследования необходимо:

Оценить наличие product/market fit у данного продукта с помощью когортного анализа, полученного на предыдущем шаге.
Пояснить свою позицию и сформулировать, на чём маркетплейс должен сконцентрироваться в ближайшее время.

# Ответ

Когортный анализ и его тепловая карта показывают, что продукт имеет низкий уровень соответствия между продуктом и рынком (product/market fit).

Уже через месяц после первой покупки сохраняется лишь 0,3% пользователей (медианный уровень удержания за первый месяц), а на второй месяц удержание падает до нуля. 
Это может свидетельствовать о недостаточной ценности (уникальности) продукта, его низком качестве или несоответствии ожиданиям клиентов — будь то сам товар, процесс оформления заказа или доставка. В результате возникает проблема с удержанием пользователей (низкий retention).

При слабом product/market fit мы не можем позволить себе рисковать масштабированием продукта на новые рынки.

Для успешного масштабирования маркетплейсу в ближайшее время необходимо сосредоточиться на повышении уровня удержания клиентов.


```python

```

# Задача 3. 

Определить 5 основных метрик, на которых продакт может сконцентрироваться, чтобы максимизировать прибыль компании.

Вы разобрались с наличием product/market fit. Теперь вас просят сформулировать продуктовые метрики маркетплейса, чтобы компания могла на них ориентироваться.

В первую очередь необходимо:

Определить 5 основных метрик, на которых продакт может сконцентрироваться, чтобы максимизировать прибыль компании.

Первая метрика должна отражать рост продаж маркетплейса.

Вторая — показывать объем аудитории, которой продукт доставляет ценность.

Третья — отражать заинтересованность новых клиентов в продакте.

Четвёртая — отражать вовлеченность клиента в продолжение использования продукта.

Пятая — отражать денежное выражение вовлеченности клиента.

Визуализируйте первую, вторую, четвёртую и пятую метрики. Используйте месячную гранулярность и созревание 1 месяц, если это нужно.

1. Рост продаж маркетплейса: Gross Merchandise Volume, GMV (Общий объем продаж)
Метрика отражает общий объем продаж через маркетплейс за определенный период. Она показывает, насколько успешно маркетплейс привлекает покупателей и генерирует доход.


```python
data_copy
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customer_id</th>
      <th>customer_unique_id</th>
      <th>customer_zip_code_prefix</th>
      <th>customer_city</th>
      <th>customer_state</th>
      <th>order_id</th>
      <th>order_status</th>
      <th>order_purchase_timestamp</th>
      <th>order_approved_at</th>
      <th>order_delivered_carrier_date</th>
      <th>...</th>
      <th>product_id</th>
      <th>seller_id</th>
      <th>shipping_limit_date</th>
      <th>price</th>
      <th>freight_value</th>
      <th>order_period</th>
      <th>order_period_day</th>
      <th>cohort</th>
      <th>cohort_day</th>
      <th>period_number_month</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>06b8999e2fba1a1fbc88172c00ba8bc7</td>
      <td>861eff4711a542e4b93843c6dd7febb0</td>
      <td>14409</td>
      <td>franca</td>
      <td>SP</td>
      <td>00e7ee1b050b8499577073aeb2a297a1</td>
      <td>delivered</td>
      <td>2017-05-16 15:05:35</td>
      <td>2017-05-16 15:22:12</td>
      <td>2017-05-23 10:47:57</td>
      <td>...</td>
      <td>a9516a079e37a9c9c36b9b78b10169e8</td>
      <td>7c67e1448b00f6e969d365cea6b010ab</td>
      <td>2017-05-22 15:22:12</td>
      <td>124.99</td>
      <td>21.88</td>
      <td>2017-05</td>
      <td>2017-05-16</td>
      <td>2017-05</td>
      <td>2017-05-16</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>18955e83d337fd6b2def6b18a428ac77</td>
      <td>290c77bc529b7ac935b93aa66c333dc3</td>
      <td>9790</td>
      <td>sao bernardo do campo</td>
      <td>SP</td>
      <td>29150127e6685892b6eab3eec79f59c7</td>
      <td>delivered</td>
      <td>2018-01-12 20:48:24</td>
      <td>2018-01-12 20:58:32</td>
      <td>2018-01-15 17:14:59</td>
      <td>...</td>
      <td>4aa6014eceb682077f9dc4bffebc05b0</td>
      <td>b8bc237ba3788b23da09c0f1f3a3288c</td>
      <td>2018-01-18 20:58:32</td>
      <td>289.00</td>
      <td>46.48</td>
      <td>2018-01</td>
      <td>2018-01-12</td>
      <td>2018-01</td>
      <td>2018-01-12</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4e7b3e00288586ebd08712fdd0374a03</td>
      <td>060e732b5b29e8181a18229c7b0b2b5e</td>
      <td>1151</td>
      <td>sao paulo</td>
      <td>SP</td>
      <td>b2059ed67ce144a36e2aa97d2c9e9ad2</td>
      <td>delivered</td>
      <td>2018-05-19 16:07:45</td>
      <td>2018-05-20 16:19:10</td>
      <td>2018-06-11 14:31:00</td>
      <td>...</td>
      <td>bd07b66896d6f1494f5b86251848ced7</td>
      <td>7c67e1448b00f6e969d365cea6b010ab</td>
      <td>2018-06-05 16:19:10</td>
      <td>139.94</td>
      <td>17.79</td>
      <td>2018-05</td>
      <td>2018-05-19</td>
      <td>2018-05</td>
      <td>2018-05-19</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>b2b6027bc5c5109e529d4dc6358b12c3</td>
      <td>259dac757896d24d7702b9acbbff3f3c</td>
      <td>8775</td>
      <td>mogi das cruzes</td>
      <td>SP</td>
      <td>951670f92359f4fe4a63112aa7306eba</td>
      <td>delivered</td>
      <td>2018-03-13 16:06:38</td>
      <td>2018-03-13 17:29:19</td>
      <td>2018-03-27 23:22:42</td>
      <td>...</td>
      <td>a5647c44af977b148e0a3a4751a09e2e</td>
      <td>7c67e1448b00f6e969d365cea6b010ab</td>
      <td>2018-03-27 16:31:16</td>
      <td>149.94</td>
      <td>23.36</td>
      <td>2018-03</td>
      <td>2018-03-13</td>
      <td>2018-03</td>
      <td>2018-03-13</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4f2d8ab171c80ec8364f7c12e35b23ad</td>
      <td>345ecd01c38d18a9036ed96c73b8d066</td>
      <td>13056</td>
      <td>campinas</td>
      <td>SP</td>
      <td>6b7d50bd145f6fc7f33cebabd7e49d0f</td>
      <td>delivered</td>
      <td>2018-07-29 09:51:30</td>
      <td>2018-07-29 10:10:09</td>
      <td>2018-07-30 15:16:00</td>
      <td>...</td>
      <td>9391a573abe00141c56e38d84d7d5b3b</td>
      <td>4a3ca9315b744ce9f8e9374361493884</td>
      <td>2018-07-31 10:10:09</td>
      <td>230.00</td>
      <td>22.25</td>
      <td>2018-07</td>
      <td>2018-07-29</td>
      <td>2018-07</td>
      <td>2018-07-29</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>112645</th>
      <td>17ddf5dd5d51696bb3d7c6291687be6f</td>
      <td>1a29b476fee25c95fbafc67c5ac95cf8</td>
      <td>3937</td>
      <td>sao paulo</td>
      <td>SP</td>
      <td>6760e20addcf0121e9d58f2f1ff14298</td>
      <td>delivered</td>
      <td>2018-04-07 15:48:17</td>
      <td>2018-04-07 16:08:45</td>
      <td>2018-04-11 02:08:36</td>
      <td>...</td>
      <td>ccb4503d9d43d245d3b295d0544f988b</td>
      <td>527801b552d0077ffd170872eb49683b</td>
      <td>2018-04-12 16:08:45</td>
      <td>74.90</td>
      <td>13.88</td>
      <td>2018-04</td>
      <td>2018-04-07</td>
      <td>2018-04</td>
      <td>2018-04-07</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>112646</th>
      <td>e7b71a9017aa05c9a7fd292d714858e8</td>
      <td>d52a67c98be1cf6a5c84435bd38d095d</td>
      <td>6764</td>
      <td>taboao da serra</td>
      <td>SP</td>
      <td>9ec0c8947d973db4f4e8dcf1fbfa8f1b</td>
      <td>delivered</td>
      <td>2018-04-04 08:20:22</td>
      <td>2018-04-04 08:35:12</td>
      <td>2018-04-05 18:42:35</td>
      <td>...</td>
      <td>9ede6b0570a75a4b9de4f383329f99ee</td>
      <td>3fd1e727ba94cfe122d165e176ce7967</td>
      <td>2018-04-10 08:35:12</td>
      <td>114.90</td>
      <td>14.16</td>
      <td>2018-04</td>
      <td>2018-04-04</td>
      <td>2018-04</td>
      <td>2018-04-04</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>112647</th>
      <td>5e28dfe12db7fb50a4b2f691faecea5e</td>
      <td>e9f50caf99f032f0bf3c55141f019d99</td>
      <td>60115</td>
      <td>fortaleza</td>
      <td>CE</td>
      <td>fed4434add09a6f332ea398efd656a5c</td>
      <td>delivered</td>
      <td>2018-04-08 20:11:50</td>
      <td>2018-04-08 20:30:03</td>
      <td>2018-04-09 17:52:17</td>
      <td>...</td>
      <td>7a5d2e1e131a860ae7d18f6fffa9d689</td>
      <td>d9e7e7778b32987280a6f2cb9a39c57d</td>
      <td>2018-04-12 20:30:03</td>
      <td>37.00</td>
      <td>19.04</td>
      <td>2018-04</td>
      <td>2018-04-08</td>
      <td>2018-04</td>
      <td>2018-04-08</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>112648</th>
      <td>56b18e2166679b8a959d72dd06da27f9</td>
      <td>73c2643a0a458b49f58cea58833b192e</td>
      <td>92120</td>
      <td>canoas</td>
      <td>RS</td>
      <td>e31ec91cea1ecf97797787471f98a8c2</td>
      <td>delivered</td>
      <td>2017-11-03 21:08:33</td>
      <td>2017-11-03 21:31:20</td>
      <td>2017-11-06 18:24:41</td>
      <td>...</td>
      <td>f819f0c84a64f02d3a5606ca95edd272</td>
      <td>4869f7a5dfa277a7dca6462dcf3b52b2</td>
      <td>2017-11-09 21:15:51</td>
      <td>689.00</td>
      <td>22.07</td>
      <td>2017-11</td>
      <td>2017-11-03</td>
      <td>2017-11</td>
      <td>2017-11-03</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>112649</th>
      <td>274fa6071e5e17fe303b9748641082c8</td>
      <td>84732c5050c01db9b23e19ba39899398</td>
      <td>6703</td>
      <td>cotia</td>
      <td>SP</td>
      <td>28db69209a75e59f20ccbb5c36a20b90</td>
      <td>delivered</td>
      <td>2017-12-19 14:27:23</td>
      <td>2017-12-19 18:50:39</td>
      <td>2017-12-21 19:17:21</td>
      <td>...</td>
      <td>017692475c1c954ff597feda05131d73</td>
      <td>3c7c4a49ec3c6550809089c6a2ca9370</td>
      <td>2017-12-26 18:50:39</td>
      <td>13.99</td>
      <td>7.78</td>
      <td>2017-12</td>
      <td>2017-12-19</td>
      <td>2017-12</td>
      <td>2017-12-19</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>110197 rows × 23 columns</p>
</div>




```python
# Агрегация по периоду заказа:
monthly_gmv = data_copy.groupby('order_period')['price'].sum().reset_index()
# Преобразование типа данных:
monthly_gmv['order_period'] = monthly_gmv['order_period'].astype(str)
# Переименование столбцов:
monthly_gmv.columns = ['order_period', 'GMV']

monthly_gmv
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_period</th>
      <th>GMV</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-09</td>
      <td>134.97</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-10</td>
      <td>40325.11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-12</td>
      <td>10.90</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-01</td>
      <td>111798.36</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-02</td>
      <td>234223.40</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-03</td>
      <td>359198.85</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-04</td>
      <td>340669.68</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-05</td>
      <td>489338.25</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-06</td>
      <td>421923.37</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-07</td>
      <td>481604.52</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017-08</td>
      <td>554699.70</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017-09</td>
      <td>607399.67</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017-10</td>
      <td>648247.65</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017-11</td>
      <td>987765.37</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2017-12</td>
      <td>726033.19</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-01</td>
      <td>924645.00</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-02</td>
      <td>826437.13</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2018-03</td>
      <td>953356.25</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2018-04</td>
      <td>973534.09</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2018-05</td>
      <td>977544.69</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018-06</td>
      <td>856077.86</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2018-07</td>
      <td>867953.46</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2018-08</td>
      <td>838576.64</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python
# Визуализация метрик
plt.figure(figsize=(10, 6))
sns.lineplot(data=monthly_gmv, x='order_period', y='GMV', marker='o')
plt.title('Рост продаж маркетплейса (GMV)')
plt.xlabel('Месяц')
plt.ylabel('GMV ($)')
plt.xticks(rotation=45)
plt.grid()
plt.tight_layout()

plt.show()
```


![png](output_34_0.png)


2. Объем аудитории, которой продукт доставляет ценность: Active User (Количество активных(платящих) клиентов) 
Метрика, отражающая число пользователей, которые совершили хотя бы одну покупку на маркетплейсе за определённый период (например, месяц). Этот показатель помогает оценить реальный объем аудитории, получающей ценность от маркетплейса, а также измерить вовлеченность и платежеспособность клиентов.


```python
# Группировка данных по временной метке заказа и подсчет уникальных пользователей
active_users = data_copy.groupby('order_purchase_timestamp')['customer_unique_id'].nunique().reset_index()
# Создание столбца с годом и месяцем покупки:
active_users['year_month'] = active_users['order_purchase_timestamp'].dt.to_period('M')
# Группировка по месяцу и суммирование уникальных пользователей:
monthly_active_users = active_users.groupby('year_month')['customer_unique_id'].sum().reset_index()
# Переименование столбцов для удобства:
monthly_active_users.columns = ['Month', 'Active Users']

monthly_active_users
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Month</th>
      <th>Active Users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-09</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-10</td>
      <td>265</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-12</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-01</td>
      <td>745</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-02</td>
      <td>1650</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-03</td>
      <td>2540</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-04</td>
      <td>2296</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-05</td>
      <td>3532</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-06</td>
      <td>3126</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-07</td>
      <td>3856</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017-08</td>
      <td>4169</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017-09</td>
      <td>4136</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017-10</td>
      <td>4467</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017-11</td>
      <td>7267</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2017-12</td>
      <td>5500</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-01</td>
      <td>7050</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-02</td>
      <td>6527</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2018-03</td>
      <td>6982</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2018-04</td>
      <td>6789</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2018-05</td>
      <td>6745</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018-06</td>
      <td>6086</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2018-07</td>
      <td>6135</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2018-08</td>
      <td>6346</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Визуализация метрик
plt.figure(figsize=(14, 10))

plt.plot(monthly_active_users['Month'].astype(str), monthly_active_users['Active Users'], marker='o', color='orange')
plt.title('Количество активных пользователей по месяцам')
plt.xlabel('Месяц')
plt.ylabel('Активные пользователи')
plt.xticks(rotation=45)
plt.grid()

plt.show()
```


![png](output_37_0.png)


3. Заинтересованность новых клиентов в продукте: Conversion to Paying User (Конверсия в первую покупку)
Метрика, показывающая долю новых пользователей, которые совершили свою первую покупку на маркетплейсе за определённый период. Этот показатель отражает степень заинтересованности новых клиентов и их готовность воспользоваться предложением, а также помогает оценить эффективность маркетинговых и продуктовых стратегий по привлечению и вовлечению аудитории.

4. Вовлеченность клиента в продолжение использования продукта: Retention Rate (Коэффициент удержания)
Метрика, отражающая процент пользователей, которые продолжают пользоваться маркетплейсрм спустя определённый период времени (например, неделю, месяц или год) после первого взаимодействия. Этот показатель помогает оценить уровень вовлеченности клиентов, их удовлетворенность маркетплейсом и способность бизнеса поддерживать долгосрочные отношения с пользователями. Высокий коэффициент удержания свидетельствует о ценности и востребованности продукта, в то время как низкий может указывать на проблемы с пользовательским опытом или конкуренцию на рынке.


```python
retention_matrix
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>period_number_month</th>
      <th>0.0</th>
      <th>1.0</th>
      <th>2.0</th>
      <th>3.0</th>
      <th>4.0</th>
      <th>5.0</th>
      <th>6.0</th>
      <th>7.0</th>
      <th>8.0</th>
      <th>9.0</th>
      <th>...</th>
      <th>11.0</th>
      <th>12.0</th>
      <th>13.0</th>
      <th>14.0</th>
      <th>15.0</th>
      <th>16.0</th>
      <th>17.0</th>
      <th>19.0</th>
      <th>20.0</th>
      <th>21.0</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-09</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>...</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>NaN</td>
      <td>0.003817</td>
      <td>0.007634</td>
      <td>0.003817</td>
      <td>0.003817</td>
    </tr>
    <tr>
      <th>2016-12</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01</th>
      <td>1.0</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>0.005579</td>
      <td>0.001395</td>
      <td>0.001395</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>0.005579</td>
      <td>0.004184</td>
      <td>0.002789</td>
      <td>NaN</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>0.002789</td>
      <td>0.001395</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02</th>
      <td>1.0</td>
      <td>0.001229</td>
      <td>0.001843</td>
      <td>0.003686</td>
      <td>0.002457</td>
      <td>0.001229</td>
      <td>0.002457</td>
      <td>0.000614</td>
      <td>0.001229</td>
      <td>0.002457</td>
      <td>...</td>
      <td>0.003686</td>
      <td>0.000614</td>
      <td>0.001843</td>
      <td>0.001229</td>
      <td>0.000614</td>
      <td>0.000614</td>
      <td>0.001843</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03</th>
      <td>1.0</td>
      <td>0.002797</td>
      <td>0.005194</td>
      <td>0.004395</td>
      <td>0.001199</td>
      <td>0.000799</td>
      <td>0.003596</td>
      <td>0.001598</td>
      <td>0.002397</td>
      <td>0.002397</td>
      <td>...</td>
      <td>0.001598</td>
      <td>0.001598</td>
      <td>0.001199</td>
      <td>0.002797</td>
      <td>0.000799</td>
      <td>0.001199</td>
      <td>0.000799</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04</th>
      <td>1.0</td>
      <td>0.003546</td>
      <td>0.002660</td>
      <td>0.000443</td>
      <td>0.002660</td>
      <td>0.003546</td>
      <td>0.002660</td>
      <td>0.004433</td>
      <td>0.002216</td>
      <td>0.002660</td>
      <td>...</td>
      <td>0.001330</td>
      <td>NaN</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>0.000887</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05</th>
      <td>1.0</td>
      <td>0.004057</td>
      <td>0.004057</td>
      <td>0.002318</td>
      <td>0.003767</td>
      <td>0.003187</td>
      <td>0.003187</td>
      <td>0.001159</td>
      <td>0.002898</td>
      <td>0.002608</td>
      <td>...</td>
      <td>0.003187</td>
      <td>0.001739</td>
      <td>0.000869</td>
      <td>0.002028</td>
      <td>0.001159</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-06</th>
      <td>1.0</td>
      <td>0.004281</td>
      <td>0.004939</td>
      <td>0.003622</td>
      <td>0.001646</td>
      <td>0.005268</td>
      <td>0.002963</td>
      <td>0.001976</td>
      <td>0.001317</td>
      <td>0.003293</td>
      <td>...</td>
      <td>0.002963</td>
      <td>0.000988</td>
      <td>0.002963</td>
      <td>0.001317</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-07</th>
      <td>1.0</td>
      <td>0.003731</td>
      <td>0.002665</td>
      <td>0.002399</td>
      <td>0.003198</td>
      <td>0.001866</td>
      <td>0.002665</td>
      <td>0.001866</td>
      <td>0.001333</td>
      <td>0.002665</td>
      <td>...</td>
      <td>0.002399</td>
      <td>0.001599</td>
      <td>0.001866</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-08</th>
      <td>1.0</td>
      <td>0.006655</td>
      <td>0.002465</td>
      <td>0.003451</td>
      <td>0.003697</td>
      <td>0.004683</td>
      <td>0.002465</td>
      <td>0.002465</td>
      <td>0.001479</td>
      <td>0.001725</td>
      <td>...</td>
      <td>0.001725</td>
      <td>0.000739</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-09</th>
      <td>1.0</td>
      <td>0.005245</td>
      <td>0.004496</td>
      <td>0.003247</td>
      <td>0.003497</td>
      <td>0.002248</td>
      <td>0.002747</td>
      <td>0.001499</td>
      <td>0.003247</td>
      <td>0.001748</td>
      <td>...</td>
      <td>0.000250</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-10</th>
      <td>1.0</td>
      <td>0.006007</td>
      <td>0.000924</td>
      <td>0.001155</td>
      <td>0.003004</td>
      <td>0.001386</td>
      <td>0.002773</td>
      <td>0.003928</td>
      <td>0.002542</td>
      <td>0.002079</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-11</th>
      <td>1.0</td>
      <td>0.003541</td>
      <td>0.002691</td>
      <td>0.001558</td>
      <td>0.001558</td>
      <td>0.001983</td>
      <td>0.001133</td>
      <td>0.001841</td>
      <td>0.000708</td>
      <td>0.000283</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-12</th>
      <td>1.0</td>
      <td>0.002623</td>
      <td>0.003559</td>
      <td>0.001873</td>
      <td>0.002997</td>
      <td>0.001686</td>
      <td>0.000937</td>
      <td>0.001124</td>
      <td>0.000937</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-01</th>
      <td>1.0</td>
      <td>0.003215</td>
      <td>0.003215</td>
      <td>0.003362</td>
      <td>0.001608</td>
      <td>0.001900</td>
      <td>0.002485</td>
      <td>0.000877</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-02</th>
      <td>1.0</td>
      <td>0.003499</td>
      <td>0.003658</td>
      <td>0.002704</td>
      <td>0.002385</td>
      <td>0.001908</td>
      <td>0.000954</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-03</th>
      <td>1.0</td>
      <td>0.003691</td>
      <td>0.001476</td>
      <td>0.002362</td>
      <td>0.001624</td>
      <td>0.000443</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-04</th>
      <td>1.0</td>
      <td>0.004558</td>
      <td>0.001671</td>
      <td>0.002279</td>
      <td>0.000912</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-05</th>
      <td>1.0</td>
      <td>0.002920</td>
      <td>0.003228</td>
      <td>0.000615</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-06</th>
      <td>1.0</td>
      <td>0.003743</td>
      <td>0.001021</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-07</th>
      <td>1.0</td>
      <td>0.001009</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2018-08</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>23 rows × 21 columns</p>
</div>




```python
# Строим тепловую карту когортного анализа
with sns.axes_style("white"):
    fig, ax = plt.subplots(1, 2, figsize=(16, 12), sharey=True, gridspec_kw={'width_ratios': [1, 11]})

    # Тепловая карта для коэффициентов удержания
    sns.heatmap(retention_matrix,
                mask=retention_matrix.isnull(),
                annot=True,
                fmt='.0%',
                cmap='RdYlGn',
                ax=ax[1])
    ax[1].set_title('Retention', fontsize=16)
    ax[1].set(xlabel='№ периода', ylabel='Когорта')

    # Тепловая карта для размеров когорт
    cohort_size_df = pd.DataFrame(cohort_size).rename(columns={0: 'cohort_size'})
    white_cmap = mcolors.ListedColormap(['white'])
    sns.heatmap(cohort_size_df,
                annot=True,
                cbar=False,
                fmt='g',
                cmap=white_cmap,
                ax=ax[0])

    fig.tight_layout()
    
    plt.show()
```


![png](output_41_0.png)


5. Денежное выражение вовлеченности клиента: Average Revenue Per Paying User, ARPPU (Средний доход на платящего пользователя)
Метрика отражает общую выручку, которую маркетплейс получает с одного клиента в среднем


```python
# Объединение данных о клиентах, заказах и GMV:
merge_data = data_copy.merge(gmv_data)
# Группировка по периоду заказа и расчет метрик:
monthly_revenue = merge_data.groupby('order_period').agg(
    total_revenue=('price', 'sum'),
    paying_users=('customer_unique_id', 'nunique')).reset_index()
# Расчет ARRPU
monthly_revenue['ARRPU'] = monthly_revenue['total_revenue'] / monthly_revenue['paying_users']
# смена формата
monthly_revenue['order_period'] = monthly_revenue['order_period'].astype(str)

monthly_revenue[['order_period', 'total_revenue', 'paying_users', 'ARRPU']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>order_period</th>
      <th>total_revenue</th>
      <th>paying_users</th>
      <th>ARRPU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-09</td>
      <td>404.91</td>
      <td>1</td>
      <td>404.910000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-10</td>
      <td>51841.06</td>
      <td>262</td>
      <td>197.866641</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-12</td>
      <td>10.90</td>
      <td>1</td>
      <td>10.900000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-01</td>
      <td>154511.67</td>
      <td>718</td>
      <td>215.197312</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-02</td>
      <td>277284.99</td>
      <td>1630</td>
      <td>170.113491</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-03</td>
      <td>431664.06</td>
      <td>2508</td>
      <td>172.114856</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-04</td>
      <td>393642.87</td>
      <td>2274</td>
      <td>173.105923</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-05</td>
      <td>601242.82</td>
      <td>3479</td>
      <td>172.820586</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-06</td>
      <td>496450.61</td>
      <td>3076</td>
      <td>161.394867</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-07</td>
      <td>601036.07</td>
      <td>3802</td>
      <td>158.084185</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017-08</td>
      <td>716108.17</td>
      <td>4114</td>
      <td>174.066157</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017-09</td>
      <td>865776.24</td>
      <td>4083</td>
      <td>212.044144</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017-10</td>
      <td>849255.89</td>
      <td>4417</td>
      <td>192.269842</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017-11</td>
      <td>1311614.89</td>
      <td>7183</td>
      <td>182.599873</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2017-12</td>
      <td>868736.71</td>
      <td>5450</td>
      <td>159.401231</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-01</td>
      <td>1167375.27</td>
      <td>6974</td>
      <td>167.389629</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-02</td>
      <td>1080191.77</td>
      <td>6400</td>
      <td>168.779964</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2018-03</td>
      <td>1199303.99</td>
      <td>6914</td>
      <td>173.460224</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2018-04</td>
      <td>1241334.54</td>
      <td>6744</td>
      <td>184.065027</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2018-05</td>
      <td>1271620.14</td>
      <td>6693</td>
      <td>189.992550</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018-06</td>
      <td>1074709.54</td>
      <td>6061</td>
      <td>177.315549</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2018-07</td>
      <td>1089799.16</td>
      <td>6100</td>
      <td>178.655600</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2018-08</td>
      <td>1025419.32</td>
      <td>6310</td>
      <td>162.507024</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Визуализация метрик
plt.figure(figsize=(10, 6))
sns.lineplot(data=monthly_revenue, x='order_period', y='ARRPU', marker='o')
plt.title('ARPPU')
plt.xlabel('Месяц')
plt.ylabel('ARPPU')
plt.xticks(rotation=45)
plt.grid()
plt.show()
```


![png](output_44_0.png)



```python

```

# Задача 4. 

Выбрать одну из 3 основных гипотез с помощью фреймворка ICE.

Посмотрев с продактом на когортный анализ и метрики, вы решили, что нужно изменить продукт. Метрики необходимо срочно повышать. Вместе с командой вы сформулировали 3 гипотезы, в которые вы верите. По каждой гипотезе команда заполнила показатели по Ease, Confidence. Вам нужно заполнить самый важный показатель — Impact. Для этого вам требуется:

Выбрать одну из трёх основных гипотез с помощью фреймворка ICE, которые были сформированы продактом и, кажется, должны улучшить пользовательский опыт в маркетплейсе.

Для расчёта Impact возьмите данные с июня 2017 года. Конверсию в повторный заказ возьмите из пункта 1 из показателя "медианный retention 1-го месяца".

Для перевода метрики в Impact воспользуйтесь следующей шкалой:


Гипотезы:

Гипотеза 1: Если исправим баг в системе процессинга заказов, то клиентам не придётся сталкиваться с проблемой отмены заказа, вследствие чего количество доставленных заказов увеличится.

Гипотеза 2: Если сократим время до отгрузки заказа, то клиенты перестанут получать свой заказ с запаздыванием, вследствие чего количество заказов увеличится за счёт повторных заказов.

Гипотеза 3: Если создадим новый способ оплаты, который будет конвертировать клиентов в повторный заказ, то клиенты не будут испытывать трудности при оформлении заказа, вследствие чего количество заказов увеличится за счёт повторных заказов.

# Гипотеза 1
Если исправим баг в системе процессинга заказов, то клиентам не придётся сталкиваться с проблемой отмены заказа, вследствие чего количество доставленных заказов увеличится. Считаем, что мы таким образом избавимся от всех отмен.


```python
Confidence_1 = 8
Ease_1 = 6
```


```python
# количество отмененных заказов с июня 2017 года
canceled = data_copy.query( 'order_purchase_timestamp > "2017-06-01 00:00:00" & order_status == "canceled"').order_id.count()
canceled
```




    0




```python
# Определяем Impact 
if canceled <= 50:
    impact = 1
elif canceled <= 150:
    impact = 2
elif canceled <= 350:
    impact = 3
elif canceled <= 750:
    impact = 4
elif canceled <= 1550:
    impact = 5
elif canceled <= 3150:
    impact = 6
elif canceled <= 6350:
    impact = 7
elif canceled <= 12750:
    impact = 8
elif canceled <= 25550:
    impact = 9
else:
    impact = 10
    
impact_1 = impact
print(f"Impact_1: {impact_1}")
```

    Impact_1: 1



```python
ICE_1 = impact * Confidence_1 * Ease_1
print(f"ICE_1: {ICE_1}")
```

    ICE_1: 48


# Гипотеза 2 
Если сократим время до отгрузки заказа, то клиенты перестанут получать свой заказ с запаздыванием, вследствие чего количество заказов увеличится за счёт повторных заказов.


```python
# Конверсию в повторный заказ (медианный retention 1-го месяца) median_retention = 0.35
Confidence_2 = 10
Ease_2 = 4
```


```python
# заказы с опозданием
orders_late= orders.query('order_purchase_timestamp > "2017-06-01 00:00:00" & order_delivered_customer_date > order_estimated_delivery_date & order_status == "delivered"')
# клиенты, получившие заказы с опозданием
customers_late = orders_late[['order_id', 'customer_id']].merge(customers[['customer_unique_id', 'customer_id']], on = 'customer_id').customer_unique_id.nunique() 
customers_late
```




    7245




```python
# клиенты, получившие заказы с опозданием с учетом конвесии 
customers = customers_late * median_retention
customers
```




    25.69148936170213




```python
# Определяем Impact 
if customers <= 50:
    impact = 1
elif customers <= 150:
    impact = 2
elif customers <= 350:
    impact = 3
elif customers <= 750:
    impact = 4
elif customers <= 1550:
    impact = 5
elif customers <= 3150:
    impact = 6
elif customers <= 6350:
    impact = 7
elif customers <= 12750:
    impact = 8
elif customers <= 25550:
    impact = 9
else:
    impact = 10
    
impact_2 = impact
print(f"Impact_2: {impact_2}")
```

    Impact_2: 1



```python
ICE_2 = impact_2 * Confidence_2 * Ease_2
print(f"ICE_2: {ICE_2}")
```

    ICE_2: 40


# Гипотеза 3
Если создадим новый способ оплаты, который будет конвертировать клиентов в повторный заказ, то клиенты не будут испытывать трудности при оформлении заказа, вследствие чего количество заказов увеличится за счёт повторных заказов.


```python
# Конверсию в повторный заказ (медианный retention 1-го месяца) median_retention = 0.35
Confidence_3 = 5
Ease_3 = 9
```


```python
# подсчитываем количество заказов для каждого клиента
cust_order_counts = data_copy[(data_copy.order_period >= '2017-06')].groupby('customer_unique_id').order_id.nunique().reset_index()

cust_order_counts.groupby('order_id').nunique('customer_unique_id')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customer_unique_id</th>
    </tr>
    <tr>
      <th>order_id</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>80558</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2137</td>
    </tr>
    <tr>
      <th>3</th>
      <td>129</td>
    </tr>
    <tr>
      <th>4</th>
      <td>23</td>
    </tr>
    <tr>
      <th>5</th>
      <td>9</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# количество клиентов, которые сделали один заказ
one_order_cust_count = (cust_order_counts == 1).order_id.sum()
one_order_cust_count
```




    80558




```python
# Возможное количество повторных заказов
repeat = one_order_cust_count * median_retention
repeat
```




    285.6666666666667




```python
# Определяем Impact 
if repeat <= 50:
    impact = 1
elif repeat <= 150:
    impact = 2
elif repeat <= 350:
    impact = 3
elif repeat <= 750:
    impact = 4
elif repeat <= 1550:
    impact = 5
elif repeat <= 3150:
    impact = 6
elif repeat <= 6350:
    impact = 7
elif repeat <= 12750:
    impact = 8
elif repeat <= 25550:
    impact = 9
else:
    impact = 10
    
impact_3 = impact
print(f"Impact_3: {impact_3}")
```

    Impact_3: 3



```python
ICE_3 = impact_3 * Confidence_3 * Ease_3
print(f"ICE_3: {ICE_3}")
```

    ICE_3: 135


# Итого  (так как ICE_1 = 192 ICE_2 = 40 ICE_3 = 135)

Гипотеза 1 (Если исправим баг в системе процессинга заказов, то клиентам не придётся сталкиваться с проблемой отмены заказа, вследствие чего количество доставленных заказов увеличится. Считаем, что мы таким образом избавимся от всех отмен.), должны улучшить пользовательский опыт в маркетплейсе.

Гипотеза 3 (Если создадим новый способ оплаты, который будет конвертировать клиентов в повторный заказ, то клиенты не будут испытывать трудности при оформлении заказа, вследствие чего количество заказов увеличится за счёт повторных заказов.) со значением ICE = 132, следующая имеет наибольший потенциал для улучшения пользовательского опыта и увеличения выручки.


```python

```

# Задача 5.

Сформулировать нужные метрики, на которые ваша гипотеза должна повлиять.

После предыдущего исследования у вас появилась гипотеза, которую можно реализовать для значительного улучшения метрик компании. Вы предложили использовать A/B-тестирование для проверки её эффективности.

Продакт попросил вас:

Сформулировать метрики, на которые должна повлиять выбранная вами гипотеза.
Сформулировать хотя бы по одной метрике в категории: целевые, прокси, guardrail.

1. Целевые метрики (Target Metrics)
Целевые метрики — это те метрики, которые напрямую отражают успех вашей гипотезы. В данном случае, мы хотим увидеть улучшение в количестве доставленных заказов и уменьшение количества отмен.

# Количество доставленных заказов (показывает общее количество заказов, которые были успешно доставлены покупателям за определенный период (например, день, месяц, квартал).

2. Прокси метрики (Proxy Metrics)
Прокси метрики — это метрики, которые могут служить индикаторами успеха целевых метрик. Они могут не быть прямыми показателями, но их изменение может указывать на изменения в целевых метриках.

# Конверсия в доставку (показывает долю заказов, которые были успешно доставлены покупателям, от общего числа оформленных заказов за определенный период.)

3. Защитные метрики (Guardrail Metrics)
Защитные метрики — это метрики, которые помогают убедиться, что изменения не приводят к негативным последствиям для бизнеса или клиентов. Они служат для мониторинга потенциальных рисков.

# Конверсия в оформление заказа (показывает процент пользователей, которые добавили товары в корзину и успешно оформили заказ. Она отражает, насколько эффективно работает процесс оформления заказа и насколько мотивированы пользователи довести покупку до конца.)

# Задача 6.

Вот и подошёл к концу ваш первый этап работы аналитиком в команде маркетплейса. Теперь необходимо поделиться результатами проведённой работы с компанией.


Сформулируйте выводы о проделанной работе, опишите их в комментариях в ipynb-файле или описании мердж-реквеста. 
Структура должна выглядеть следующим образом:
   - формализация проблемы продукта;

   - описание выводов из пункта 1;

   - описание выводов из пункта 2;

   - описание выводов из пункта 3;

   - описание выводов из пункта 4;

   - описание выводов из пункта 5;

   - общие выводы по итогу исследования;

   - рекомендации по продукту.

# Вывод и рекомендации:

# Формализация проблемы продукта: 
    
    Отсутсвует рост выручки

    
    


# Описание выводов из пункта 1:
    
    После проведение кагортного анализа мы выяснили, что уровень Retention – менее 1% клиентов совершают повторные покупки.
    Выручка поддерживается за счет привлечения новых пользователей.

# Описание выводов из пункта 2:
    
    Из пункта 1 следует, что у маркетплейса слабый product/market fit. Это может свидетельствовать о низкой ценности или уникальности продукта, несоответствии ожиданиям клиентов, а также проблемах с качеством товаров, процессом оформления заказов или доставкой.
    
    При отсутствии устойчивого product/market fit масштабирование на новые рынки несет значительные риски. Чтобы обеспечить успешное развитие, маркетплейсу необходимо в первую очередь сосредоточиться на повышении удержания клиентов.
    
    

# Описание выводов из пункта 3:
    
    Ключевые метрики для максимизации прибыли маркетплейса:
        
    GMV (Общий объем продаж)
    Active User (Количество активных(платящих) клиентов)
    Conversion to Paying User (Конверсия в первую покупку)
    Retention Rate (Коэффициент удержания)
    ARPPU (Средний доход на платящего пользователя)
    


# Описание выводов из пункта 4:
    
    Проверка гипотез с использованием фреймворка ICE показал, что наибольшее влияние на рост ключевых метрик окажут изменения, предложенные в гипотезе 1 (Если исправим баг в системе процессинга заказов, то клиентам не придётся сталкиваться с проблемой отмены заказа, вследствие чего количество доставленных заказов увеличится. Считаем, что мы таким образом избавимся от всех отмен.), так как ее ICE = 192.
    Исправление бага в системе процессинга заказов устранит проблему отмен заказов, что приведет к увеличению количества успешно доставленных заказов. Это, в свою очередь, повысит удовлетворенность клиентов, увеличит Retention за счет снижения негативного опыта пользователей, а также положительно повлияет на выручку и ARPPU. В долгосрочной перспективе исправление этой ошибки может снизить переменные расходы, связанные с возвратами и обработкой отмен.

# Описание выводов из пункта 5:
    
    Принятая нами гипотеза 1 повлияет на следующие метрики:
        
    Целевая: Количество доставленных заказов (показывает общее количество заказов, которые были успешно доставлены  покупателям за определенный период)
        
    Прокси:  Конверсия в доставку (показывает долю заказов, которые были успешно доставлены покупателям, от общего  числа оформленных заказов за определенный период.)
        
    Защитная метрика: Конверсия в оформление заказа (показывает процент пользователей, которые добавили товары в    корзину и успешно оформили заказ. Она отражает, насколько эффективно работает процесс оформления заказа и           насколько мотивированы пользователи довести покупку до конца.)




# Общие выводы по итогу исследования: 
    
    Отсутствие роста выручки связано с низким уровнем Retention – менее 1% клиентов совершают повторные покупки. Выручка поддерживается за счет привлечения новых пользователей, что указывает на отсутствие product/market fit, а значит, масштабирование пока невозможно. Основной фокус должен быть на повышении возврата клиентов.
    Итого без улучшения Retention рост выручки останется нестабильным. Улучшение удержания клиентов станет драйвером роста всех ключевых метрик.

# Рекомендации по продукту:
    
        
Первоночально устранить баг в системе процессинга заказов, который приводит к отменам. Это положительно повлияет на доход, прибыль, ARPU, CSAT и Retention.
Также маркетплейсу необходимо выстраивать долгосрочные отношения с клиентами, обеспечивая качественный сервис, персонализированные предложения и удобный процесс покупки. Это поможет снизить отток пользователей и повысить повторные покупки.

    



```python

```


```python

```


```python

```


```python

```


```python

```


```python

```
