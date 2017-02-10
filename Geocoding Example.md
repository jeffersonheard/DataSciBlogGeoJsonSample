

```python
import pandas as pd
import numpy as np
from geopy.geocoders import Nominatim
import usaddress
import re
import json
```

First thing we need to do is load the dataset. Pandas conveniently understands URLs, so we can load them directly from the website.


```python
# If you're running this on your own on a fast connection, this will work. 
# I recommend writing out the dataframe later.
# tax_bills_june15_bbls = pd.read_csv("http://taxbills.nyc/tax_bills_june15_bbls.csv")
# tax_bills_june15_exab = pd.read_csv("http://taxbills.nyc/tax_bills_june15_exab.csv")

# loading these over HTTP turns out to be an arduous task
tax_bills_june15_bbls = pd.read_csv("/Users/jeff/Downloads/tax_bills_june15_bbls.csv", index_col='bbl')
tax_bills_june15_exab = pd.read_csv("/Users/jeff/Downloads/tax_bills_june15_exab.csv", index_col='bbl')

```


```python
tax_bills_june15_bbls
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ownername</th>
      <th>address</th>
      <th>taxclass</th>
      <th>taxrate</th>
      <th>emv</th>
      <th>tbea</th>
      <th>bav</th>
      <th>tba</th>
      <th>propertytax</th>
      <th>condonumber</th>
      <th>condo</th>
    </tr>
    <tr>
      <th>bbl</th>
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
      <th>1000010010</th>
      <td>GOVERNORS ISLAND CORPORATION</td>
      <td>GOVERNORS ISLAND CORPORATION\nC/O TRUST FOR. G...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>337672000.0</td>
      <td>15749050.0</td>
      <td>147407802.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000010101</th>
      <td>U. S. GOVT LAND &amp; BLDGS</td>
      <td>BEDLOES ISLAND\n1 LIBERTY ISLAND\nELLIS ISLAND,</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>25607000.0</td>
      <td>1106496.0</td>
      <td>10356570.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000010201</th>
      <td>U. S. GOVT LAND &amp; BLDGS</td>
      <td>ELLIS ISLAND\n1 LIBERTY ISLAND\nELLIS ISLAND,</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>233982000.0</td>
      <td>10366655.0</td>
      <td>97029720.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000020001</th>
      <td>NYC DSBS</td>
      <td>NYC DSBS\n110 WILLIAM ST. FL. 7\nNEW YORK , NY...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>69458000.0</td>
      <td>3163690.0</td>
      <td>29611473.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000020002</th>
      <td>10 SSA LANDLORD, LLC</td>
      <td>10 SSA LANDLORD, LLC\n729 7TH AVE. FL. 15\nNEW...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>55592000.0</td>
      <td>2672762.0</td>
      <td>25016491.0</td>
      <td>654246.0</td>
      <td>654246.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000020003</th>
      <td>NOT ON FILE</td>
      <td>\nBAD LOCATION ADDRESS\n,</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>1774000.0</td>
      <td>83277.0</td>
      <td>779458.0</td>
      <td>83277.0</td>
      <td>83277.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000020023</th>
      <td>NYC DSBS</td>
      <td>NYC DSBS\n110 WILLIAM ST. FL. 7\nNEW YORK , NY...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>36968000.0</td>
      <td>1824996.0</td>
      <td>17081581.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000030001</th>
      <td>PARKS AND RECREATION (GENERAL)</td>
      <td>PARKS AND RECREATION (GENERAL)\nARSENAL WEST\n...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>285745000.0</td>
      <td>13749587.0</td>
      <td>128693250.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000030002</th>
      <td>PARKS AND RECREATION (GENERAL)</td>
      <td>PARKS AND RECREATION (GENERAL)\nARSENAL WEST\n...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10918000.0</td>
      <td>524916.0</td>
      <td>4913100.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000030003</th>
      <td>PARKS AND RECREATION (GENERAL)</td>
      <td>PARKS AND RECREATION (GENERAL)\nARSENAL WEST\n...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>9484000.0</td>
      <td>432000.0</td>
      <td>4043430.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000030010</th>
      <td>UNITED STATES AMERICA</td>
      <td>UNITED STATES AMERICA\n26 FEDERAL PLZ. STE 30-...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>32516000.0</td>
      <td>1440628.0</td>
      <td>13483980.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000041001</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>5351542.0</td>
      <td>254361.0</td>
      <td>2380764.0</td>
      <td>254361.0</td>
      <td>251840.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041002</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>7733995.0</td>
      <td>367600.0</td>
      <td>3440656.0</td>
      <td>367600.0</td>
      <td>342763.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041003</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>15960040.0</td>
      <td>713000.0</td>
      <td>6673528.0</td>
      <td>713000.0</td>
      <td>713000.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041004</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>1372802.0</td>
      <td>65249.0</td>
      <td>610721.0</td>
      <td>65249.0</td>
      <td>53869.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041005</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>3144677.0</td>
      <td>149467.0</td>
      <td>1398982.0</td>
      <td>149467.0</td>
      <td>148035.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041006</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>2973077.0</td>
      <td>141311.0</td>
      <td>1322645.0</td>
      <td>141311.0</td>
      <td>141311.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041007</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10487584.0</td>
      <td>498479.0</td>
      <td>4665659.0</td>
      <td>498479.0</td>
      <td>384952.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041008</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN, LLC\n16220 N. SCOT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041009</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nRYAN LLC\n16220 N. SCOTT...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041010</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041011</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041012</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041013</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041014</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10208233.0</td>
      <td>485201.0</td>
      <td>4541380.0</td>
      <td>485201.0</td>
      <td>374697.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041015</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10204251.0</td>
      <td>485012.0</td>
      <td>4539612.0</td>
      <td>485012.0</td>
      <td>374508.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041016</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10072551.0</td>
      <td>478752.0</td>
      <td>4481020.0</td>
      <td>478752.0</td>
      <td>369737.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041017</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10072551.0</td>
      <td>478752.0</td>
      <td>4481020.0</td>
      <td>478752.0</td>
      <td>478752.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041018</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10399795.0</td>
      <td>494306.0</td>
      <td>4626605.0</td>
      <td>494306.0</td>
      <td>494306.0</td>
      <td>835.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1000041019</th>
      <td>ONE NY PLAZA CO. LLC</td>
      <td>ONE NY PLAZA CO. LLC\nC/O RYAN DEPT. 113\nP.O....</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>10399795.0</td>
      <td>494306.0</td>
      <td>4626605.0</td>
      <td>494306.0</td>
      <td>494306.0</td>
      <td>835.0</td>
      <td>unit</td>
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
    </tr>
    <tr>
      <th>5080500001</th>
      <td>NUSSER- MEANY, SUSAN M.</td>
      <td>NUSSER- MEANY, SUSAN M.\nSTATEN ISLAND, NY 103...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>591000.0</td>
      <td>5754.0</td>
      <td>30038.0</td>
      <td>5754.0</td>
      <td>5754.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500004</th>
      <td>JEANNE GIORLANDO</td>
      <td>JEANNE GIORLANDO\n7631 AMBOY RD.\nSTATEN ISLAN...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>533000.0</td>
      <td>5420.0</td>
      <td>28290.0</td>
      <td>5118.0</td>
      <td>5118.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500007</th>
      <td>A. MARKOWITZ</td>
      <td>A. MARKOWITZ\n7635 AMBOY RD.\nSTATEN ISLAND, N...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>572000.0</td>
      <td>5656.0</td>
      <td>29527.0</td>
      <td>4975.0</td>
      <td>4975.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500010</th>
      <td>CHARLES BEARDSLEY</td>
      <td>CHARLES BEARDSLEY\n7639 AMBOY RD.\nSTATEN ISLA...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>465000.0</td>
      <td>3432.0</td>
      <td>17914.0</td>
      <td>2926.0</td>
      <td>2926.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500013</th>
      <td>JOHN ALLIDA SCOTTI</td>
      <td>JOHN ALLIDA SCOTTI\n7647 AMBOY RD.\nSTATEN ISL...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>633000.0</td>
      <td>6177.0</td>
      <td>32246.0</td>
      <td>5875.0</td>
      <td>5875.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500017</th>
      <td>NUSSER, STACEY</td>
      <td>NUSSER, STACEY\n7688 AMBOY RD.\nSTATEN ISLAND,...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>474000.0</td>
      <td>5448.0</td>
      <td>28440.0</td>
      <td>5448.0</td>
      <td>5448.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500019</th>
      <td>BECKETT, JOSEPH</td>
      <td>BECKETT, JOSEPH\nSTATEN ISLAND, NY 10307-1418\...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>497000.0</td>
      <td>5282.0</td>
      <td>27570.0</td>
      <td>4980.0</td>
      <td>4980.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500022</th>
      <td>PAUL A. MANDILE</td>
      <td>PAUL A. MANDILE\n7663 AMBOY RD.\nSTATEN ISLAND...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>867000.0</td>
      <td>9345.0</td>
      <td>48781.0</td>
      <td>9043.0</td>
      <td>9043.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500025</th>
      <td>ALAN J. OLSEN</td>
      <td>ALAN J. OLSEN\n7671 AMBOY RD.\nSTATEN ISLAND, ...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>457000.0</td>
      <td>4847.0</td>
      <td>25303.0</td>
      <td>4545.0</td>
      <td>4545.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500028</th>
      <td>CLAIRE DOGERY</td>
      <td>CLAIRE DOGERY\nSTATEN ISLAND, NY 10307-1418\nO...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>400000.0</td>
      <td>4512.0</td>
      <td>23554.0</td>
      <td>1501.0</td>
      <td>1501.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500031</th>
      <td>STEFANIE DIMINO</td>
      <td>STEFANIE DIMINO\nSTATEN ISLAND, NY 10307-1418\...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>490000.0</td>
      <td>5324.0</td>
      <td>27793.0</td>
      <td>5324.0</td>
      <td>5324.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500034</th>
      <td>V. SCHNURR</td>
      <td>V. SCHNURR\n590 CRAIG AVE.\nSTATEN ISLAND, NY ...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>497000.0</td>
      <td>5148.0</td>
      <td>26872.0</td>
      <td>4222.0</td>
      <td>4222.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500037</th>
      <td>ANTONIELLO, FRANK</td>
      <td>ANTONIELLO, FRANK\nSTATEN ISLAND, NY 10307-123...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>570000.0</td>
      <td>6552.0</td>
      <td>34200.0</td>
      <td>6552.0</td>
      <td>6552.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500050</th>
      <td>VAUGHAN, BRIAN</td>
      <td>VAUGHAN, BRIAN\n582 CRAIG AVE.\nSTATEN ISLAND,...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>613000.0</td>
      <td>5979.0</td>
      <td>31213.0</td>
      <td>5677.0</td>
      <td>5677.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500053</th>
      <td>OGNO, CHRISTOPHER E.</td>
      <td>OGNO, CHRISTOPHER E.\nSTATEN ISLAND, NY 10307-...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>553000.0</td>
      <td>6324.0</td>
      <td>33009.0</td>
      <td>6324.0</td>
      <td>6324.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500055</th>
      <td>NOT ON FILE</td>
      <td>\nBAD LOCATION ADDRESS\n,</td>
      <td>1b - vacant land, zoned residential</td>
      <td>19.1570%</td>
      <td>6000.0</td>
      <td>25.0</td>
      <td>129.0</td>
      <td>25.0</td>
      <td>25.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500056</th>
      <td>WYSOCZNSKI, ANDRE</td>
      <td>MR. &amp; MRS. ANDRZEJ WYSOCZANSKI\nMONROE, NJ 088...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>472000.0</td>
      <td>4463.0</td>
      <td>23296.0</td>
      <td>4463.0</td>
      <td>4463.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500058</th>
      <td>CATHERINE MARINO</td>
      <td>CATHERINE MARINO\nSTATEN ISLAND, NY 10307-1237...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>488000.0</td>
      <td>5273.0</td>
      <td>27523.0</td>
      <td>4971.0</td>
      <td>4971.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500060</th>
      <td>COMO, PIETRO</td>
      <td>COMO, PIETRO\nSTATEN ISLAND, NY 10307-1237\nOu...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>622000.0</td>
      <td>5424.0</td>
      <td>28315.0</td>
      <td>5424.0</td>
      <td>5424.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500062</th>
      <td>JOHN K. CHAPMAN</td>
      <td>JOHN K. CHAPMAN\n560 CRAIG AVE.\nSTATEN ISLAND...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>486000.0</td>
      <td>5201.0</td>
      <td>27147.0</td>
      <td>4899.0</td>
      <td>4899.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500065</th>
      <td>BROWN, ELIZABETH</td>
      <td>BROWN, ELIZABETH\nLONG BRANCH, NJ 07740-4932\n...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>492000.0</td>
      <td>5417.0</td>
      <td>28279.0</td>
      <td>5417.0</td>
      <td>5417.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500068</th>
      <td>MARIO AMOROSO</td>
      <td>MARIO AMOROSO\nSTATEN ISLAND, NY 10307-1237\nO...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>550000.0</td>
      <td>5974.0</td>
      <td>31185.0</td>
      <td>5672.0</td>
      <td>5672.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500072</th>
      <td>JOSEPH R. GLORIA</td>
      <td>JOSEPH R. GLORIA\n536 CRAIG AVE.\nSTATEN ISLAN...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>668000.0</td>
      <td>7258.0</td>
      <td>37887.0</td>
      <td>6956.0</td>
      <td>6956.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500076</th>
      <td>DENNIS EMPEROR</td>
      <td>DENNIS EMPEROR\n534 CRAIG AVE.\nSTATEN ISLAND,...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>442000.0</td>
      <td>4701.0</td>
      <td>24538.0</td>
      <td>3801.0</td>
      <td>3801.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500078</th>
      <td>CHIU, ANNE</td>
      <td>CHIU, ANNE\n532 CRAIG AVE.\nSTATEN ISLAND, NY ...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>1033000.0</td>
      <td>10413.0</td>
      <td>54355.0</td>
      <td>10413.0</td>
      <td>10413.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500083</th>
      <td>TOBIN, GALE</td>
      <td>TOBIN, GALE\n142 BENTLEY ST.\nSTATEN ISLAND, N...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>475000.0</td>
      <td>5361.0</td>
      <td>27986.0</td>
      <td>5059.0</td>
      <td>5059.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500086</th>
      <td>ARLOTTA, THOMAS</td>
      <td>ARLOTTA, THOMAS\nSTATEN ISLAND, NY 10307-1235\...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>585000.0</td>
      <td>3432.0</td>
      <td>17914.0</td>
      <td>3130.0</td>
      <td>3130.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500089</th>
      <td>JOHN GERVASI</td>
      <td>JOHN GERVASI\nSTATEN ISLAND, NY 10307-1235\nOu...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>507000.0</td>
      <td>5282.0</td>
      <td>27570.0</td>
      <td>2020.0</td>
      <td>2020.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500092</th>
      <td>RITA M. MOOG</td>
      <td>WILLIAM P. MOOG\nSTATEN ISLAND, NY 10307-1235\...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>484000.0</td>
      <td>5296.0</td>
      <td>27644.0</td>
      <td>4994.0</td>
      <td>4994.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5080500094</th>
      <td>EDWARD DONOHUE</td>
      <td>EDWARD DONOHUE\n162 BENTLEY ST.\nSTATEN ISLAND...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>448000.0</td>
      <td>5148.0</td>
      <td>26872.0</td>
      <td>4846.0</td>
      <td>4846.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>1081624 rows × 11 columns</p>
</div>




```python
tax_bills_june15_exab
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>detail</th>
      <th>amount</th>
      <th>units</th>
    </tr>
    <tr>
      <th>bbl</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4015180009</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-4075.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4046020125</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-11794.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4001570040</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-6942.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4004740010</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-3548.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4012820175</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-8735.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4096480024</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-9421.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4008811001</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-9702.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4066880010</th>
      <td>exemption</td>
      <td>clergy</td>
      <td>-287.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4022201059</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-69.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4114171507</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-106.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5064220040</th>
      <td>exemption</td>
      <td>park</td>
      <td>-1218.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5077020043</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-9906.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5009530307</th>
      <td>exemption</td>
      <td>park</td>
      <td>-1139.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5065770038</th>
      <td>exemption</td>
      <td>park</td>
      <td>-1483.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3025970001</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-72473.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3002751203</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-1286.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3068020036</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-2567.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3067160075</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-1438.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3078780009</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-684.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3021111197</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-10772.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3008050016</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-12288.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3024770001</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-97241.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3067820050</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-2253.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3002451578</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-503.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3073970001</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-8575.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3067210070</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-1875.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3065030030</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-10109.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3053390001</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-3833.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3006650071</th>
      <td>exemption</td>
      <td>clergy</td>
      <td>-287.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2039441082</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-775.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2057630556</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630554</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630546</th>
      <td>exemption</td>
      <td>enhanced star - school tax relief</td>
      <td>-621.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630540</th>
      <td>exemption</td>
      <td>senior citizens homeowners’ exemption</td>
      <td>-3338.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630540</th>
      <td>exemption</td>
      <td>enhanced star - school tax relief</td>
      <td>-621.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630565</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630563</th>
      <td>exemption</td>
      <td>disabled homeowner</td>
      <td>-3003.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630563</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630533</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2057630525</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060703</th>
      <td>exemption</td>
      <td>faculty student hsg</td>
      <td>-12139.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060721</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060708</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060681</th>
      <td>exemption</td>
      <td>school-elem,hs,acad</td>
      <td>-196908.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060698</th>
      <td>exemption</td>
      <td>school-elem,hs,acad</td>
      <td>-16195.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2058060723</th>
      <td>exemption</td>
      <td>basic star - school tax relief</td>
      <td>-302.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2023280032</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-2337.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2023280017</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-14241.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2023280035</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-84983.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1014440041</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-4792.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1018381116</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-59.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1018333209</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-138.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1012061087</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-417.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1001971110</th>
      <td>exemption</td>
      <td>icip</td>
      <td>-8178.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1020901029</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-20.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1014201524</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-294.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1018261122</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-951.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1016021056</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-448.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1020421124</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-3547.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1012500006</th>
      <td>abatement</td>
      <td>j51 abatement</td>
      <td>-7042.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>752599 rows × 4 columns</p>
</div>



Note that address isn't exactly right. We'll have to normalize it a bit before we go further.  First thing to do is get rid of the pesky newlines.  It turns out they're not really newlines, since we read a CSV, but rather literal escapes. 

My first thought was to simply get rid of them as in below:


```python
corrected_addresses = tax_bills_june15_bbls['address'].str.replace("\\\\n", ' ')
next(iter(corrected_addresses))
```




    'GOVERNORS ISLAND CORPORATION C/O TRUST FOR. GOVERNORS ISLAN 10 SOUTH ST. APT. FRNT SLIP7'



But it turns out that a lot of geocoders don't respond well to recipient names at the head of the address, zip4s, and borough names instead of city names. Our solution won't be perfect, but it'll be decent enough to show as an example and leave "perfection" for the reader if it's really necessary for the application.  

Nominatim in particular is fragile, but we use it because it's free and requires no API key.  We also use it because in order to do bulk geocoding you're going to have to set up Nominatim yourself or another equally fragile bulk geocoder (there are several).

So let's go ahead and do the corrections.

First we'll correct some common quirks. We'll drop empty addresses and replace "One" with 1, which is common in commercial districts.


```python
# starts_with_number = re.compile('^[0-9]+')
starts_with_one = re.compile('^ONE')

def scrub_addr(addr_lines):
    global starts_with_number
    if not isinstance(addr_lines, list):
        return addr_lines
    
    if starts_with_one.match(addr_lines[0]):  # this is a common pattern in addresses
        addr_lines[0] = addr_lines[0].replace('ONE', '1')
        
    if len(addr_lines):
        return ' '.join(addr_lines)
    else:
        return np.nan
```

This next function will take the addresses we ahve and make sure they at least have state and zipcode attached.


```python
state_and_zip = re.compile(r'^.*NY\s+(?:[0-9]{5})?(?:-[0-9]{4})\s*(?:USA)?$')

def append_state_info(addr):
    if not state_and_zip.match(addr):
        return addr + ', NY 00000'  # we append a dummy zipcode because it helps the address tagger work better.
    else:
        return addr
```

Now that we've cleaned up common quirks, let's tag the address. Rule number 1. Addresses, despite being ubiquitous, are _messy_. **There is an (likely open-source) address parser for your country. Find it and use it. Don't try to create your own**. Writing your own address parser will cause damage to your monitor, keyboard, desk, and face.  We don't want that, do we? 

My code is using `usaddress`.  It's slow but accurate, so we'll not actually be doing the entire dataset in this notebook.  You can of course do the whole dataset yourself without trouble, but you should really go make some pancakes from scratch and eat them while you're waiting for it to finish.


```python
def tag_addr(addr):
    if not isinstance(addr, str) or len(addr) == 0:
        return np.nan
    else:
        try:
            return usaddress.tag(addr)
        except usaddress.RepeatedLabelError:  # some addresses turn out to have bits repeated
            return tag_addr(' '.join(addr.split(' ')[1:]))
```

A final pass will re-merge the address into a single string for geocoding, keeping only the parts of the address that the geocoder understands. 


```python
def join_addr(addr):
    addr = addr[0]
    if 'ZipCode' in addr and '-' in addr['ZipCode']:
        addr['ZipCode'] = addr['ZipCode'].split('-')[0]   # Nominatim hates zip4
        
    if 'PlaceName' not in addr:
        addr['PlaceName'] = 'New York'  # we already know we're in new york, some addresses omit it.
    
    if all((
        'AddressNumber' in addr,
        'StreetName' in addr,
        'ZipCode' in addr and addr['ZipCode'] != '00000'
    )):
        return ' '.join((
                addr.get('AddressNumberPrefix', ''),
                addr.get('AddressNumber', ''),
                addr.get('AddressNumberSuffix', ''),
                addr.get('StreetNamePreModifier', ''),
                addr.get('StreetNamePreDirectional', ''),
                addr.get('StreetNamePreType', ''),
                addr.get('StreetName', ''),
                addr.get('StreetNamePostType', ''),
                addr.get('StreetNamePostDirectional', ''),
                addr['StateName'], 
                addr['ZipCode']
            ))
    elif all((
        'AddressNumber' in addr,
        'StreetName' in addr
    )):
        return ' '.join((
            addr.get('AddressNumberPrefix', ''),
            addr.get('AddressNumber', ''),
            addr.get('AddressNumberSuffix', ''),
            addr.get('StreetNamePreModifier', ''),
            addr.get('StreetNamePreDirectional', ''),
            addr.get('StreetNamePreType', ''),
            addr.get('StreetName', ''),
            addr.get('StreetNamePostType', ''),
            addr.get('StreetNamePostDirectional', ''),
            addr['StateName'],     
        ))
    else:
        return np.nan
```


```python
    
scrubbed_addresses = tax_bills_june15_bbls['address']\
    .sample(n=150)\
    .str.split("\\\\n")\
    .map(scrub_addr)\
    .dropna()\
    .map(append_state_info)\
    .map(tag_addr)\
    .map(join_addr)\
    .dropna()

len(scrubbed_addresses)
```




    104




```python
geocodable_tax_bills_june15_bbls = tax_bills_june15_bbls.ix[scrubbed_addresses.index]
geocodable_tax_bills_june15_bbls['address'] = scrubbed_addresses
geocodable_tax_bills_june15_bbls
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ownername</th>
      <th>address</th>
      <th>taxclass</th>
      <th>taxrate</th>
      <th>emv</th>
      <th>tbea</th>
      <th>bav</th>
      <th>tba</th>
      <th>propertytax</th>
      <th>condonumber</th>
      <th>condo</th>
    </tr>
    <tr>
      <th>bbl</th>
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
      <th>4050221346</th>
      <td>WINSTON TOWER, LLC</td>
      <td>315     CENTRAL PARK  W. NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>47791.0</td>
      <td>2532.0</td>
      <td>19698.0</td>
      <td>2532.0</td>
      <td>2532.0</td>
      <td>162.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>5022680027</th>
      <td>ALAN NANCY EILENBERG</td>
      <td>228     LONDON RD.  NY 10306</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>714000.0</td>
      <td>8066.0</td>
      <td>42103.0</td>
      <td>7764.0</td>
      <td>7764.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4011690028</th>
      <td>TZIKAS, NICK (TRUSTEE)</td>
      <td>3242     74TH ST.  NY 11370</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>591000.0</td>
      <td>5902.0</td>
      <td>30811.0</td>
      <td>5600.0</td>
      <td>5600.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1000151174</th>
      <td>NEKKAH, KEVIN ERWIN</td>
      <td>20     WEST ST.  NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>165429.0</td>
      <td>8485.0</td>
      <td>66004.0</td>
      <td>3072.0</td>
      <td>2074.0</td>
      <td>1557.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>4001391039</th>
      <td>AMINOV, BARNO</td>
      <td>16427     75TH RD.  NY 11366</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>43144.0</td>
      <td>2349.0</td>
      <td>18274.0</td>
      <td>2349.0</td>
      <td>2349.0</td>
      <td>646.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3053400039</th>
      <td>CHAMBERS, DESMOND</td>
      <td>246   E.  8TH ST.  NY 11218</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>877000.0</td>
      <td>5058.0</td>
      <td>26403.0</td>
      <td>4756.0</td>
      <td>4756.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4072510031</th>
      <td>WOISLAVSKY, IRWIN</td>
      <td>18442     TUDOR RD.  NY 11432</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>976000.0</td>
      <td>9572.0</td>
      <td>49968.0</td>
      <td>9270.0</td>
      <td>9270.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4132000004</th>
      <td>PATEL, JYOTI</td>
      <td>13560     234TH PL.  NY 11422</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>446000.0</td>
      <td>4298.0</td>
      <td>22438.0</td>
      <td>4298.0</td>
      <td>4298.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2038730049</th>
      <td>GUERRA, LOUIS J.</td>
      <td>1345     ROSEDALE AVE.  NY 10472</td>
      <td>2a - 4-6 unit residential building</td>
      <td>12.8550%</td>
      <td>448000.0</td>
      <td>6261.0</td>
      <td>48708.0</td>
      <td>5645.0</td>
      <td>5645.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5040330032</th>
      <td>GARY S. SCALESCI</td>
      <td>69     CUBA AVE.  NY 10306</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>257000.0</td>
      <td>2954.0</td>
      <td>15420.0</td>
      <td>2652.0</td>
      <td>2652.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2038060039</th>
      <td>HAGANS, DARRYL</td>
      <td>2162     CHATTERTON AVE.  NY 10472</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>395000.0</td>
      <td>4242.0</td>
      <td>22144.0</td>
      <td>3940.0</td>
      <td>3940.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2054230006</th>
      <td>SCALA PROPERTIES INC.</td>
      <td>40     GRANDVIEW CIR.  NY 11030</td>
      <td>2a - 4-6 unit residential building</td>
      <td>12.8550%</td>
      <td>667000.0</td>
      <td>7991.0</td>
      <td>62162.0</td>
      <td>7991.0</td>
      <td>7991.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4093920028</th>
      <td>KHAN JAMALODEEN</td>
      <td>9419     108TH ST.  NY 11419</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>460000.0</td>
      <td>4162.0</td>
      <td>21728.0</td>
      <td>3860.0</td>
      <td>3860.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5054940083</th>
      <td>YNCLINO CYNTHIA</td>
      <td>42     CORTELYOU AVE.  NY 10312</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>392000.0</td>
      <td>4380.0</td>
      <td>22863.0</td>
      <td>4078.0</td>
      <td>4078.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3047270062</th>
      <td>PAUL BASTIEN</td>
      <td>329   E.  55TH ST.  NY</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>365000.0</td>
      <td>4090.0</td>
      <td>21349.0</td>
      <td>1424.0</td>
      <td>1424.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5000480018</th>
      <td>YORK-VANDUZER, LLC</td>
      <td>150     GREAVES LN.  NY 10308</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>290000.0</td>
      <td>2245.0</td>
      <td>11721.0</td>
      <td>2245.0</td>
      <td>2245.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2042481003</th>
      <td>2013 COLONIAL LLC</td>
      <td>2013     COLONIAL LLC 69 BOLTON AVE.  NY 10605</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>61169.0</td>
      <td>3587.0</td>
      <td>27903.0</td>
      <td>72.0</td>
      <td>72.0</td>
      <td>124.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1011651703</th>
      <td>CARTUS FINANCIAL CORPORATION</td>
      <td>2109     BROADWAY   NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>426152.0</td>
      <td>21451.0</td>
      <td>166867.0</td>
      <td>21451.0</td>
      <td>17697.0</td>
      <td>799.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3032490012</th>
      <td>NAVARRO, LEONOR</td>
      <td>1638     DEKALB AVE.  NY 11237</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>568000.0</td>
      <td>3457.0</td>
      <td>18047.0</td>
      <td>3155.0</td>
      <td>3155.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4114441016</th>
      <td>LOZITO, MARIANA</td>
      <td>15614     76TH ST.  NY 11414</td>
      <td>1a - condo unit in 1-3 story building</td>
      <td>19.1570%</td>
      <td>299841.0</td>
      <td>2258.0</td>
      <td>11786.0</td>
      <td>2258.0</td>
      <td>2176.0</td>
      <td>59.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3048730036</th>
      <td>BERTRAND MERISE</td>
      <td>3615     CHURCH AVE.  NY 11203</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>556000.0</td>
      <td>6391.0</td>
      <td>33360.0</td>
      <td>6089.0</td>
      <td>6089.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2039442458</th>
      <td>DENNIS MOHABIR</td>
      <td>1972     POWELL AVE.  NY 10472</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>65086.0</td>
      <td>3150.0</td>
      <td>24504.0</td>
      <td>1204.0</td>
      <td>142.0</td>
      <td>2.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3057940014</th>
      <td>EVERBRIGHT BROOKLYN LLC</td>
      <td>714     61ST ST.  NY 11220</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>327000.0</td>
      <td>14203.0</td>
      <td>132935.0</td>
      <td>14203.0</td>
      <td>14203.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5036420053</th>
      <td>ZARRILLI DANIEL</td>
      <td>136     BACHE AVE.  NY 10306</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>491000.0</td>
      <td>5239.0</td>
      <td>27348.0</td>
      <td>4937.0</td>
      <td>4937.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2047520059</th>
      <td>RODRIQUES, VALERIE ROSE</td>
      <td>3211     WICKHAM AVE.  NY 10469</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>453000.0</td>
      <td>5207.0</td>
      <td>27180.0</td>
      <td>4905.0</td>
      <td>4905.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3071010006</th>
      <td>CHEN, WEN NAN</td>
      <td>232    AVENUE T.   NY 11223</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>961000.0</td>
      <td>7270.0</td>
      <td>37951.0</td>
      <td>3014.0</td>
      <td>3014.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4045800027</th>
      <td>SPYRO AVDOULOS</td>
      <td>16012     11TH AVE.  NY 11357</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>999000.0</td>
      <td>8647.0</td>
      <td>45136.0</td>
      <td>8647.0</td>
      <td>8647.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4097270072</th>
      <td>YEH, SUMI CHUANG</td>
      <td>15015     87TH AVE.  NY 11432</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>603000.0</td>
      <td>5879.0</td>
      <td>30688.0</td>
      <td>5577.0</td>
      <td>5577.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3043050052</th>
      <td>COLE-WRIGHT, DORIAN</td>
      <td>713     VAN SICLEN AVE.  NY 11207</td>
      <td>2a - 4-6 unit residential building</td>
      <td>12.8550%</td>
      <td>130000.0</td>
      <td>6747.0</td>
      <td>52488.0</td>
      <td>6747.0</td>
      <td>6747.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5053630040</th>
      <td>VALERIO CHIRONNA</td>
      <td>188     SEIDMAN AVE.  NY 10312</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>494000.0</td>
      <td>5678.0</td>
      <td>29640.0</td>
      <td>2060.0</td>
      <td>2060.0</td>
      <td>NaN</td>
      <td>NaN</td>
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
    </tr>
    <tr>
      <th>1006401015</th>
      <td>BONN INVESTMENT, INC.</td>
      <td>400   W.  12TH ST.  NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>786133.0</td>
      <td>42910.0</td>
      <td>333797.0</td>
      <td>18214.0</td>
      <td>18214.0</td>
      <td>2060.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>5017150001</th>
      <td>GOETHALS SOUTH LLC</td>
      <td>TX     75088-5526   NY</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>1130000.0</td>
      <td>53703.0</td>
      <td>502650.0</td>
      <td>49376.0</td>
      <td>49376.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5047130031</th>
      <td>WILLIAM C. MEANEY</td>
      <td>205     FAIRBANKS AVE.  NY 10306</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>529000.0</td>
      <td>6019.0</td>
      <td>31418.0</td>
      <td>5717.0</td>
      <td>5717.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4033110022</th>
      <td>YU, LING</td>
      <td>11729     UNION TPKE.  NY 11375</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>1069000.0</td>
      <td>7097.0</td>
      <td>37047.0</td>
      <td>6795.0</td>
      <td>6795.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5026200037</th>
      <td>CARMELITA DE JESUS</td>
      <td>68     TOWERS LN.  NY 10314</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>362000.0</td>
      <td>4161.0</td>
      <td>21720.0</td>
      <td>3859.0</td>
      <td>3859.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4155950024</th>
      <td>EVERTON ANGUS</td>
      <td>611     JARVIS AVE.  NY 11691</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>431000.0</td>
      <td>4905.0</td>
      <td>25603.0</td>
      <td>4905.0</td>
      <td>4905.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3007730065</th>
      <td>323 LUOS PROPERTY LLC</td>
      <td>133     MOTT ST.  NY 10013</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>658000.0</td>
      <td>2178.0</td>
      <td>11370.0</td>
      <td>2178.0</td>
      <td>2178.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3074590059</th>
      <td>1517 VOORHIES AVE. LLC</td>
      <td>1517     VOORHIES AVE.  NY 11235</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>2156000.0</td>
      <td>77646.0</td>
      <td>726750.0</td>
      <td>6423.0</td>
      <td>6423.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4163000004</th>
      <td>ATHENA BENDO</td>
      <td>14309     NEWPORT AVE.  NY 11694</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>1006000.0</td>
      <td>11563.0</td>
      <td>60360.0</td>
      <td>11563.0</td>
      <td>11563.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3010271092</th>
      <td>ZHENG, YAN XIA</td>
      <td>110     MADISON ST.  NY 10002</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>65055.0</td>
      <td>3235.0</td>
      <td>25164.0</td>
      <td>91.0</td>
      <td>91.0</td>
      <td>2786.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>1021080039</th>
      <td>515 EDGECOMBE OWNERS CP</td>
      <td>343     SAINT NICHOLAS AVE.  NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>1720000.0</td>
      <td>81248.0</td>
      <td>632035.0</td>
      <td>79738.0</td>
      <td>64497.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3048900015</th>
      <td>LEESEAP REID</td>
      <td>270   E.  37TH ST.  NY 11203</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>360000.0</td>
      <td>3902.0</td>
      <td>20366.0</td>
      <td>3600.0</td>
      <td>3600.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3067961005</th>
      <td>WHITMAN PUISANG CHOI</td>
      <td>1720   E.  14TH ST.  NY 11229</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>58500.0</td>
      <td>2906.0</td>
      <td>22604.0</td>
      <td>2604.0</td>
      <td>2604.0</td>
      <td>250.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3033440152</th>
      <td>RODRIGUEZ, LORENZO</td>
      <td>305     PALMETTO ST.  NY 11237</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>540000.0</td>
      <td>4485.0</td>
      <td>23414.0</td>
      <td>2329.0</td>
      <td>2329.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4125230072</th>
      <td>SHARPLIS-ESPRIT, LUCIA</td>
      <td>17451     128TH AVE.  NY 11434</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>433000.0</td>
      <td>4736.0</td>
      <td>24720.0</td>
      <td>4736.0</td>
      <td>4736.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4032410055</th>
      <td>VAY MIN HOM</td>
      <td>7107     NANSEN ST.  NY 11375</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>648000.0</td>
      <td>4871.0</td>
      <td>25428.0</td>
      <td>1814.0</td>
      <td>1814.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1001421296</th>
      <td>MARLINCA PROPERTIES LIMITED</td>
      <td>200     CHAMBERS ST.  NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>169209.0</td>
      <td>9515.0</td>
      <td>74015.0</td>
      <td>6142.0</td>
      <td>6142.0</td>
      <td>1629.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>2039443852</th>
      <td>FERNANDES, DANIELLE LOFF</td>
      <td>1641     METROPOLITAN AVE.  NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>55677.0</td>
      <td>2714.0</td>
      <td>21110.0</td>
      <td>1024.0</td>
      <td>119.0</td>
      <td>2.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>2047370021</th>
      <td>CHARLES ALLEN</td>
      <td>3339     FENTON AVE.  NY 10469</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>389000.0</td>
      <td>4471.0</td>
      <td>23340.0</td>
      <td>4169.0</td>
      <td>4169.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4122740011</th>
      <td>EARL MERCURIUS</td>
      <td>16107     137TH AVE.  NY 11434</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>582000.0</td>
      <td>5909.0</td>
      <td>30846.0</td>
      <td>5607.0</td>
      <td>5607.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4011690008</th>
      <td>JANILY LEE TRUST</td>
      <td>7316     32ND AVE.  NY 11370</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>594000.0</td>
      <td>6011.0</td>
      <td>31380.0</td>
      <td>5709.0</td>
      <td>5709.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4068970003</th>
      <td>TSANG, WINNIE</td>
      <td>16915     65TH AVE.  NY 11365</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>708000.0</td>
      <td>6781.0</td>
      <td>35395.0</td>
      <td>6479.0</td>
      <td>6479.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2039090001</th>
      <td>PMC LLC</td>
      <td>320     WEST ST.  NY 10528</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>527000.0</td>
      <td>24251.0</td>
      <td>226980.0</td>
      <td>5433.0</td>
      <td>5433.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3082850069</th>
      <td>THOMAS, JULIUS, RUTHY</td>
      <td>1360   E.  102ND ST.  NY 11236</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>424000.0</td>
      <td>4874.0</td>
      <td>25440.0</td>
      <td>4572.0</td>
      <td>4572.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3078570043</th>
      <td>MASSIE, OLYMPHISE</td>
      <td>1209   E.  59TH ST.  NY 11234</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>316000.0</td>
      <td>3632.0</td>
      <td>18960.0</td>
      <td>1195.0</td>
      <td>1195.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4150100052</th>
      <td>145-47 155TH ST. REALTY</td>
      <td>1392     BEECH ST.  NY 11509</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>590000.0</td>
      <td>22529.0</td>
      <td>210870.0</td>
      <td>5847.0</td>
      <td>5847.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4082200097</th>
      <td>RIZO, HERMAN</td>
      <td>5110     MARATHON PKWY.  NY 11362</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>699000.0</td>
      <td>6905.0</td>
      <td>36045.0</td>
      <td>6603.0</td>
      <td>6603.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4069860022</th>
      <td>JEAN WONG</td>
      <td>7513     167TH ST.  NY 11366</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>584000.0</td>
      <td>5165.0</td>
      <td>26960.0</td>
      <td>5165.0</td>
      <td>5165.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4023720143</th>
      <td>ROSE ALGOZINI</td>
      <td>5315     62ND ST.  NY 11378</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>549000.0</td>
      <td>5323.0</td>
      <td>27786.0</td>
      <td>5323.0</td>
      <td>5323.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4090470017</th>
      <td>KHAIR, SIKDAR M.</td>
      <td>9722     76TH ST.  NY 11416</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>503000.0</td>
      <td>4559.0</td>
      <td>23797.0</td>
      <td>4559.0</td>
      <td>4559.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 11 columns</p>
</div>



Now these look good.  However, geocoding all of them with `geopy` will prove impossible. Not only will it take until sometime next year to complete because of rate limiting, but you will find that geocoding services like to charge a fee for their services beyond a certain number of records. In awhile we'll show you how to load your own geocoder and use it.

For now, we'll take a subset.  Some services rate limit you to one call a second.  To avoid taking forever, we'll just use a few records at first.


```python
# grab a random sample of records
tax_bills_bbls_sample = geocodable_tax_bills_june15_bbls.sample(n=15)

# grab the same records from the key-linked dataframe.
tax_bills_exab_sample = tax_bills_june15_exab.ix[tax_bills_bbls_sample.index]
```


```python
tax_bills_bbls_sample
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ownername</th>
      <th>address</th>
      <th>taxclass</th>
      <th>taxrate</th>
      <th>emv</th>
      <th>tbea</th>
      <th>bav</th>
      <th>tba</th>
      <th>propertytax</th>
      <th>condonumber</th>
      <th>condo</th>
    </tr>
    <tr>
      <th>bbl</th>
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
      <th>4006480018</th>
      <td>RABOS, CONSTANTINE</td>
      <td>3157  35TH ST. NY 11106</td>
      <td>2a - 4-6 unit residential building</td>
      <td>12.8550%</td>
      <td>818000.0</td>
      <td>12720.0</td>
      <td>98946.0</td>
      <td>12720.0</td>
      <td>12720.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4134860062</th>
      <td>WILSON, MARJORIE</td>
      <td>14536  230TH ST. NY 11413</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>494000.0</td>
      <td>5020.0</td>
      <td>26203.0</td>
      <td>4718.0</td>
      <td>4718.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4081360040</th>
      <td>DENNIS DELORENZO</td>
      <td>4132  WESTMORELAND ST. NY 11363</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>1350000.0</td>
      <td>9973.0</td>
      <td>52058.0</td>
      <td>9671.0</td>
      <td>9671.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3042680035</th>
      <td>JAMES OXLEY FAMILY IRREVOCABLE TRUST</td>
      <td>664  HEMLOCK ST. NY 11208</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>440000.0</td>
      <td>4735.0</td>
      <td>24715.0</td>
      <td>4433.0</td>
      <td>4433.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3053990052</th>
      <td>MELVIN BRICKMAN</td>
      <td>507  F.  NY 11218</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>929000.0</td>
      <td>7885.0</td>
      <td>41158.0</td>
      <td>7583.0</td>
      <td>7583.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3001070024</th>
      <td>PARKS AND RECREATION (GENERAL)</td>
      <td>16 61ST ST. New York NY</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>182000.0</td>
      <td>8471.0</td>
      <td>79290.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3073330067</th>
      <td>FRANKIE KAFAI LAU</td>
      <td>2053 E. 28TH ST. NY 11229</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>408000.0</td>
      <td>4690.0</td>
      <td>24480.0</td>
      <td>4388.0</td>
      <td>4388.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4104270034</th>
      <td>PATRICK, GLORIA</td>
      <td>18833  KEESEVILLE AVE. NY 11412</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>450000.0</td>
      <td>3761.0</td>
      <td>19634.0</td>
      <td>3459.0</td>
      <td>3459.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1006440063</th>
      <td>FAIRFAX &amp; SAMMONS PROPERTIES, LLC</td>
      <td>67  GANSEVOORT ST. NY 10014</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>4157000.0</td>
      <td>157150.0</td>
      <td>1470890.0</td>
      <td>157150.0</td>
      <td>157150.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4082470008</th>
      <td>ALICE PONEROS</td>
      <td>25123  THEBES AVE. NY 11362</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>841000.0</td>
      <td>7523.0</td>
      <td>39269.0</td>
      <td>6902.0</td>
      <td>6902.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4157090005</th>
      <td>GRETEL JOSEPH</td>
      <td>2210  LORETTA RD. NY 11691</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>371000.0</td>
      <td>4264.0</td>
      <td>22260.0</td>
      <td>3962.0</td>
      <td>3962.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1008701214</th>
      <td>CHEN, ADRIAN</td>
      <td>1 IRVING PLACE New York NY</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>286232.0</td>
      <td>13722.0</td>
      <td>106748.0</td>
      <td>13722.0</td>
      <td>13722.0</td>
      <td>449.0</td>
      <td>unit</td>
    </tr>
    <tr>
      <th>3032750036</th>
      <td>JANICE GEIGER WATSON</td>
      <td>96  HIMROD ST. NY 11221</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>588000.0</td>
      <td>1782.0</td>
      <td>9304.0</td>
      <td>1480.0</td>
      <td>1480.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2033630067</th>
      <td>ROBERT MAUCH</td>
      <td>4222  HERKIMER PL. NY 10470</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>457000.0</td>
      <td>5253.0</td>
      <td>27420.0</td>
      <td>4320.0</td>
      <td>4320.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4114174401</th>
      <td>GIAQUINTO GINA</td>
      <td>8710  149TH AVE. NY 11414</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>55425.0</td>
      <td>3033.0</td>
      <td>23596.0</td>
      <td>2731.0</td>
      <td>1964.0</td>
      <td>12.0</td>
      <td>unit</td>
    </tr>
  </tbody>
</table>
</div>



Now a quick sanity check to make sure our addresses work.


```python
print(next(iter(tax_bills_bbls_sample['address'])))
geocoder = Nominatim()
geocoder.geocode(next(iter(tax_bills_bbls_sample['address'])))

```

    3157  35TH ST. NY 11106





    Location(35th Street, Astoria, Queens County, NYC, New York, 11101, United States of America, (40.7628959, -73.9202263, 0.0))




```python
geocoded_addresses = tax_bills_bbls_sample['address'].map(lambda addr: geocoder.geocode(addr))
geocoded_addresses
```




    bbl
    4006480018    (35th Street, Astoria, Queens County, NYC, New...
    4134860062    (230th Street, Laurelton, Queens County, NYC, ...
    4081360040    (Westmoreland Street, Douglaston, Queens Count...
    3042680035    (664, Hemlock Street, East New York, Kings Cou...
    3053990052    (507, Avenue F, Parkville, BK, Kings County, N...
    3001070024    (60-16, 61st Street, Fresh Pond, Queens County...
    3073330067    (2053, East 28th Street, Sheepshead Bay, BK, K...
    4104270034    (Keeseville Avenue, Saint Albans, Queens Count...
    1006440063    (67, Gansevoort Street, Chelsea, Manhattan, Ne...
    4082470008    (Thebes Avenue, Little Neck, Queens County, NY...
    4157090005    (Loretta Road, Roy Reuther Houses, Far Rockawa...
    1008701214    (72 1/2, Irving Place, Flatiron, Manhattan, Ne...
    3032750036    (96, Himrod Street, Bushwick, Kings County, NY...
    2033630067    (4222, Herkimer Place, Woodlawn, Bronx, Bronx ...
    4114174401    (149th Avenue, Ozone Park, Kings, NYC, New Yor...
    Name: address, dtype: object



Now we have some geocoded addresses! We got lucky here, but in a larger sample, some will be `None`. A `.dropna()` or filter should suffice to get rid of null values. Each location will have a bunch of fields, but the most useful are the canonical address, longitude, and latitude:


```python
loc = next(iter(geocoded_addresses))  # let's inspect the first one.
print((loc.address, (loc.longitude, loc.latitude)))
```

    ('35th Street, Astoria, Queens County, NYC, New York, 11101, United States of America', (-73.9202263, 40.7628959))


Now the only thing left is to add Series for these to our sample.  I like to canonicalize the address, too, although you may want to rename the original address field to `original_address` or some such: 


```python
tax_bills_bbls_sample['address'] = geocoded_addresses.map(lambda l: l.address)
tax_bills_bbls_sample['latitude'] = geocoded_addresses.map(lambda l: l.latitude)
tax_bills_bbls_sample['longitude'] = geocoded_addresses.map(lambda l: l.longitude)
tax_bills_bbls_sample
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ownername</th>
      <th>address</th>
      <th>taxclass</th>
      <th>taxrate</th>
      <th>emv</th>
      <th>tbea</th>
      <th>bav</th>
      <th>tba</th>
      <th>propertytax</th>
      <th>condonumber</th>
      <th>condo</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
    <tr>
      <th>bbl</th>
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
      <th>4006480018</th>
      <td>RABOS, CONSTANTINE</td>
      <td>35th Street, Astoria, Queens County, NYC, New ...</td>
      <td>2a - 4-6 unit residential building</td>
      <td>12.8550%</td>
      <td>818000.0</td>
      <td>12720.0</td>
      <td>98946.0</td>
      <td>12720.0</td>
      <td>12720.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.762896</td>
      <td>-73.920226</td>
    </tr>
    <tr>
      <th>4134860062</th>
      <td>WILSON, MARJORIE</td>
      <td>230th Street, Laurelton, Queens County, NYC, N...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>494000.0</td>
      <td>5020.0</td>
      <td>26203.0</td>
      <td>4718.0</td>
      <td>4718.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.659682</td>
      <td>-73.750916</td>
    </tr>
    <tr>
      <th>4081360040</th>
      <td>DENNIS DELORENZO</td>
      <td>Westmoreland Street, Douglaston, Queens County...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>1350000.0</td>
      <td>9973.0</td>
      <td>52058.0</td>
      <td>9671.0</td>
      <td>9671.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.772826</td>
      <td>-73.738065</td>
    </tr>
    <tr>
      <th>3042680035</th>
      <td>JAMES OXLEY FAMILY IRREVOCABLE TRUST</td>
      <td>664, Hemlock Street, East New York, Kings Coun...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>440000.0</td>
      <td>4735.0</td>
      <td>24715.0</td>
      <td>4433.0</td>
      <td>4433.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.672613</td>
      <td>-73.868584</td>
    </tr>
    <tr>
      <th>3053990052</th>
      <td>MELVIN BRICKMAN</td>
      <td>507, Avenue F, Parkville, BK, Kings County, NY...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>929000.0</td>
      <td>7885.0</td>
      <td>41158.0</td>
      <td>7583.0</td>
      <td>7583.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.633838</td>
      <td>-73.973400</td>
    </tr>
    <tr>
      <th>3001070024</th>
      <td>PARKS AND RECREATION (GENERAL)</td>
      <td>60-16, 61st Street, Fresh Pond, Queens County,...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>182000.0</td>
      <td>8471.0</td>
      <td>79290.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.714971</td>
      <td>-73.902534</td>
    </tr>
    <tr>
      <th>3073330067</th>
      <td>FRANKIE KAFAI LAU</td>
      <td>2053, East 28th Street, Sheepshead Bay, BK, Ki...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>408000.0</td>
      <td>4690.0</td>
      <td>24480.0</td>
      <td>4388.0</td>
      <td>4388.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.601201</td>
      <td>-73.943775</td>
    </tr>
    <tr>
      <th>4104270034</th>
      <td>PATRICK, GLORIA</td>
      <td>Keeseville Avenue, Saint Albans, Queens County...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>450000.0</td>
      <td>3761.0</td>
      <td>19634.0</td>
      <td>3459.0</td>
      <td>3459.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.698718</td>
      <td>-73.766175</td>
    </tr>
    <tr>
      <th>1006440063</th>
      <td>FAIRFAX &amp; SAMMONS PROPERTIES, LLC</td>
      <td>67, Gansevoort Street, Chelsea, Manhattan, New...</td>
      <td>4 - commercial property</td>
      <td>10.6840%</td>
      <td>4157000.0</td>
      <td>157150.0</td>
      <td>1470890.0</td>
      <td>157150.0</td>
      <td>157150.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.739595</td>
      <td>-74.007467</td>
    </tr>
    <tr>
      <th>4082470008</th>
      <td>ALICE PONEROS</td>
      <td>Thebes Avenue, Little Neck, Queens County, NYC...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>841000.0</td>
      <td>7523.0</td>
      <td>39269.0</td>
      <td>6902.0</td>
      <td>6902.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.766735</td>
      <td>-73.734254</td>
    </tr>
    <tr>
      <th>4157090005</th>
      <td>GRETEL JOSEPH</td>
      <td>Loretta Road, Roy Reuther Houses, Far Rockaway...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>371000.0</td>
      <td>4264.0</td>
      <td>22260.0</td>
      <td>3962.0</td>
      <td>3962.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.603080</td>
      <td>-73.756490</td>
    </tr>
    <tr>
      <th>1008701214</th>
      <td>CHEN, ADRIAN</td>
      <td>72 1/2, Irving Place, Flatiron, Manhattan, New...</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>286232.0</td>
      <td>13722.0</td>
      <td>106748.0</td>
      <td>13722.0</td>
      <td>13722.0</td>
      <td>449.0</td>
      <td>unit</td>
      <td>40.736727</td>
      <td>-73.986599</td>
    </tr>
    <tr>
      <th>3032750036</th>
      <td>JANICE GEIGER WATSON</td>
      <td>96, Himrod Street, Bushwick, Kings County, NYC...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>588000.0</td>
      <td>1782.0</td>
      <td>9304.0</td>
      <td>1480.0</td>
      <td>1480.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.696093</td>
      <td>-73.923497</td>
    </tr>
    <tr>
      <th>2033630067</th>
      <td>ROBERT MAUCH</td>
      <td>4222, Herkimer Place, Woodlawn, Bronx, Bronx C...</td>
      <td>1 - small home, less than 4 families</td>
      <td>19.1570%</td>
      <td>457000.0</td>
      <td>5253.0</td>
      <td>27420.0</td>
      <td>4320.0</td>
      <td>4320.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40.896346</td>
      <td>-73.875535</td>
    </tr>
    <tr>
      <th>4114174401</th>
      <td>GIAQUINTO GINA</td>
      <td>149th Avenue, Ozone Park, Kings, NYC, New York...</td>
      <td>2 - residential, more than 10 units</td>
      <td>12.8550%</td>
      <td>55425.0</td>
      <td>3033.0</td>
      <td>23596.0</td>
      <td>2731.0</td>
      <td>1964.0</td>
      <td>12.0</td>
      <td>unit</td>
      <td>40.668572</td>
      <td>-73.856841</td>
    </tr>
  </tbody>
</table>
</div>




```python
output = []
import json
for i, row in tax_bills_bbls_sample.iterrows():
    feature = {
        "type": "Feature", 
        "geometry": {"type": "Point", "coordinates": [row['longitude'], row['latitude']]},
        "properties": {}
    }
    for key, value in row.items():
        if not pd.isnull(value):
            feature['properties'][key] = value
    output.append(feature)

with open('SampleFeatures.')
print(json.dumps(output, indent=2))
```

    [
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.9202263,
            40.7628959
          ]
        },
        "properties": {
          "emv": 818000.0,
          "ownername": "RABOS, CONSTANTINE",
          "bav": 98946.0,
          "tba": 12720.0,
          "taxclass": "2a - 4-6 unit residential building",
          "tbea": 12720.0,
          "propertytax": 12720.0,
          "address": "35th Street, Astoria, Queens County, NYC, New York, 11101, United States of America",
          "taxrate": "12.8550%",
          "longitude": -73.9202263,
          "latitude": 40.7628959
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.7509159,
            40.659682
          ]
        },
        "properties": {
          "emv": 494000.0,
          "ownername": "WILSON, MARJORIE",
          "bav": 26203.0,
          "tba": 4718.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 5020.0,
          "propertytax": 4718.0,
          "address": "230th Street, Laurelton, Queens County, NYC, New York, 11413, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.7509159,
          "latitude": 40.659682
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.7380649,
            40.772826
          ]
        },
        "properties": {
          "emv": 1350000.0,
          "ownername": "DENNIS DELORENZO",
          "bav": 52058.0,
          "tba": 9671.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 9973.0,
          "propertytax": 9671.0,
          "address": "Westmoreland Street, Douglaston, Queens County, NYC, New York, 11363, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.7380649,
          "latitude": 40.772826
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.8685837397059,
            40.6726126
          ]
        },
        "properties": {
          "emv": 440000.0,
          "ownername": "JAMES OXLEY FAMILY IRREVOCABLE TRUST",
          "bav": 24715.0,
          "tba": 4433.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 4735.0,
          "propertytax": 4433.0,
          "address": "664, Hemlock Street, East New York, Kings County, NYC, New York, 11208, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.8685837397059,
          "latitude": 40.6726126
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.9733999834947,
            40.63383805
          ]
        },
        "properties": {
          "emv": 929000.0,
          "ownername": "MELVIN BRICKMAN",
          "bav": 41158.0,
          "tba": 7583.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 7885.0,
          "propertytax": 7583.0,
          "address": "507, Avenue F, Parkville, BK, Kings County, NYC, New York, 11218, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.9733999834947,
          "latitude": 40.63383805
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.9025341058374,
            40.7149715
          ]
        },
        "properties": {
          "emv": 182000.0,
          "ownername": "PARKS AND RECREATION (GENERAL)",
          "bav": 79290.0,
          "taxclass": "4 - commercial property",
          "tbea": 8471.0,
          "propertytax": 0.0,
          "address": "60-16, 61st Street, Fresh Pond, Queens County, NYC, New York, 11378, United States of America",
          "taxrate": "10.6840%",
          "longitude": -73.9025341058374,
          "latitude": 40.7149715
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.94377505,
            40.60120065
          ]
        },
        "properties": {
          "emv": 408000.0,
          "ownername": "FRANKIE KAFAI LAU",
          "bav": 24480.0,
          "tba": 4388.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 4690.0,
          "propertytax": 4388.0,
          "address": "2053, East 28th Street, Sheepshead Bay, BK, Kings County, NYC, New York, 11229, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.94377505,
          "latitude": 40.60120065
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.7661746,
            40.6987185
          ]
        },
        "properties": {
          "emv": 450000.0,
          "ownername": "PATRICK, GLORIA",
          "bav": 19634.0,
          "tba": 3459.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 3761.0,
          "propertytax": 3459.0,
          "address": "Keeseville Avenue, Saint Albans, Queens County, NYC, New York, 11412, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.7661746,
          "latitude": 40.6987185
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -74.0074671751662,
            40.7395952
          ]
        },
        "properties": {
          "emv": 4157000.0,
          "ownername": "FAIRFAX & SAMMONS PROPERTIES, LLC",
          "bav": 1470890.0,
          "tba": 157150.0,
          "taxclass": "4 - commercial property",
          "tbea": 157150.0,
          "propertytax": 157150.0,
          "address": "67, Gansevoort Street, Chelsea, Manhattan, New York County, NYC, New York, 10014, United States of America",
          "taxrate": "10.6840%",
          "longitude": -74.0074671751662,
          "latitude": 40.7395952
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.7342539,
            40.766735
          ]
        },
        "properties": {
          "emv": 841000.0,
          "ownername": "ALICE PONEROS",
          "bav": 39269.0,
          "tba": 6902.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 7523.0,
          "propertytax": 6902.0,
          "address": "Thebes Avenue, Little Neck, Queens County, NYC, New York, 11362, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.7342539,
          "latitude": 40.766735
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.7564903,
            40.6030799
          ]
        },
        "properties": {
          "emv": 371000.0,
          "ownername": "GRETEL JOSEPH",
          "bav": 22260.0,
          "tba": 3962.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 4264.0,
          "propertytax": 3962.0,
          "address": "Loretta Road, Roy Reuther Houses, Far Rockaway, Queens County, NYC, New York, 11691, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.7564903,
          "latitude": 40.6030799
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.9865989,
            40.7367272
          ]
        },
        "properties": {
          "emv": 286232.0,
          "condo": "unit",
          "bav": 106748.0,
          "tbea": 13722.0,
          "propertytax": 13722.0,
          "tba": 13722.0,
          "ownername": "CHEN, ADRIAN",
          "latitude": 40.7367272,
          "condonumber": 449.0,
          "taxclass": " 2 - residential, more than 10 units",
          "address": "72 1/2, Irving Place, Flatiron, Manhattan, New York County, NYC, New York, 10003, United States of America",
          "taxrate": "12.8550%",
          "longitude": -73.9865989
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.92349675,
            40.69609325
          ]
        },
        "properties": {
          "emv": 588000.0,
          "ownername": "JANICE GEIGER WATSON",
          "bav": 9304.0,
          "tba": 1480.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 1782.0,
          "propertytax": 1480.0,
          "address": "96, Himrod Street, Bushwick, Kings County, NYC, New York, 11221, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.92349675,
          "latitude": 40.69609325
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.8755345223434,
            40.8963464
          ]
        },
        "properties": {
          "emv": 457000.0,
          "ownername": "ROBERT MAUCH",
          "bav": 27420.0,
          "tba": 4320.0,
          "taxclass": " 1 - small home, less than 4 families",
          "tbea": 5253.0,
          "propertytax": 4320.0,
          "address": "4222, Herkimer Place, Woodlawn, Bronx, Bronx County, NYC, New York, 10470, United States of America",
          "taxrate": "19.1570%",
          "longitude": -73.8755345223434,
          "latitude": 40.8963464
        },
        "type": "Feature"
      },
      {
        "geometry": {
          "type": "Point",
          "coordinates": [
            -73.8568412,
            40.6685722
          ]
        },
        "properties": {
          "emv": 55425.0,
          "condo": "unit",
          "bav": 23596.0,
          "tbea": 3033.0,
          "propertytax": 1964.0,
          "tba": 2731.0,
          "ownername": "GIAQUINTO GINA",
          "latitude": 40.6685722,
          "condonumber": 12.0,
          "taxclass": " 2 - residential, more than 10 units",
          "address": "149th Avenue, Ozone Park, Kings, NYC, New York, 11414; 11208, United States of America",
          "taxrate": "12.8550%",
          "longitude": -73.8568412
        },
        "type": "Feature"
      }
    ]



```python

```
