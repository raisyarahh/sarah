# Boston-Crime

# -*- coding: utf-8 -*-
"""
Created on Thu Mar 5 21:51:59 2020
@author: user
"""
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
data = pd.read_csv('Boston Crime 2015-2019.csv')
#dilakukan pengecekan missing data
data.isnull().sum()
#mengisi blank space dengan nan
data1=data.replace(r'^\s*$', np.nan, regex=True)
#menghapus kolom shooting karena terdapat 425073 missing data
del data1['SHOOTING']
#menghapus missing data, karena data yang hilang tidak dapat di rata-rata-kan untuk mengganti dengan nilai rata-rata
data2=data1.dropna()
#menghapus data duplicate
data3=data2.drop_duplicates()
#Pengecekan missing data kembali
data3.isnull().sum()
#Trend kejadian per hari
import seaborn as sns
data4 = data3.groupby(['DAY_OF_WEEK'])['OFFENSE_CODE'].count().reset_index()
fig = plt.figure(figsize=(12,8))
ax = fig.add_subplot(111)
p=sns.lineplot(x=data4.iloc[:,0], y=data4.iloc[:,1], data=data4)
p.set_ylabel('No. of Crimes Occurred')
p.set_xlabel('Day Of Week')
plt.tight_layout()
plt.show()
#Trend masing-masing kejadian per tahun
data5= pd.DataFrame(data3.groupby(['YEAR','OFFENSE_CODE_GROUP']).size().sort_values(ascending=False).rename('COUNT').reset_index())
plt.figure(figsize=(15, 18))
ax = sns.lineplot(x='YEAR',
y='COUNT',
hue='OFFENSE_CODE_GROUP',
data=data5).legend(loc='center left',
bbox_to_anchor=(1, 0.5),
fancybox=True,
shadow=True,
borderpad=1)
#Komposisi kejadian berdasarkan banyaknya kejadian
fig = plt.figure(figsize=(12,12))
ax = fig.add_subplot(111)
d = data3['OFFENSE_CODE_GROUP'].value_counts().reset_index()
p = sns.barplot(x=d.iloc[:,1], y=d.iloc[:,0], data=d, palette='summer')
p.set_xlabel('No. of crimes occurred')
p.set_ylabel('Offense Code group')
plt.tight_layout()
plt.show()
#Proporsi
labels = data3['OFFENSE_CODE_GROUP'].astype('category').cat.categories.tolist()
counts = data3['OFFENSE_CODE_GROUP'].value_counts()
sizes = [counts[var_cat] for var_cat in labels]
fig1, ax1 = plt.subplots(figsize = (22,12))
ax1.pie(sizes, labels=labels, autopct='%1.1f%%', shadow=True, startangle=140)
ax1.axis('equal')
plt.show()
#Kejadian per jam
sns.catplot(x='HOUR',
kind='count',
height=8.27,
aspect=3,
color='green',
data=data3)
plt.xticks(size=30)
plt.yticks(size=30)
plt.xlabel('HOUR', fontsize=40)
plt.ylabel('Count', fontsize=40)
#Kejadian per hari
sns.catplot(x='DAY_OF_WEEK',
kind='count',
height=8,
aspect=3,
data=data)
plt.xticks(size=30)
plt.yticks(size=30)
plt.xlabel('')
plt.ylabel('Count', fontsize=40)
#Kejadian per bulan
months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']
sns.catplot(x='MONTH',
kind='count',
height=8,
aspect=3,
color='green',
data=data3)
plt.xticks(np.arange(12), months, size=30)
plt.yticks(size=30)
plt.xlabel('')
plt.ylabel('Count', fontsize=40)
#Korelasi map
f,ax = plt.subplots(figsize=(10, 10))
sns.heatmap(data3.corr(), annot=True, linewidths=.7, fmt= '.1f',ax=ax)
plt.show()
#Data korelasi dengan hari libur
import datetime
data3['OCCURRED_ON_DATE'] = pd.to_datetime(data3['OCCURRED_ON_DATE'])
cpd = data3.set_index('OCCURRED_ON_DATE')
cpd18 = cpd[cpd['YEAR'] == 2018]
crime18 = pd.DataFrame(cpd18.resample("D").size())
fig, ax = plt.subplots(figsize=(20, 10))
ax.plot(crime18[[0]], label="CPD")
ax.annotate("Independence Day", xy=('2018-7-4', 250), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(10, -40), textcoords='offset points', ha='center',
arrowprops=dict(arrowstyle="->"))
ax.annotate("New Year's Day", xy=('2018-1-1', 220), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(10, -40), textcoords='offset points', ha='left',
arrowprops=dict(arrowstyle="->"))
ax.annotate("Labor Day", xy=('2018-9-4', 220), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(10, -40), textcoords='offset points', ha='left',
arrowprops=dict(arrowstyle="->"))
ax.annotate("Halloween", xy=('2018-10-31', 270), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(-80, -50), textcoords='offset points', ha='left',
arrowprops=dict(arrowstyle="->"))
ax.annotate("Thanksgiving", xy=('2018-11-25', 150), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(-80, -40), textcoords='offset points', ha='left',
arrowprops=dict(arrowstyle="->"))
ax.annotate("Christmas", xy=('2018-12-25', 150), xycoords='data',size=15,
bbox=dict(boxstyle="round", fc="none", ec="gray"),
xytext=(10, -40), textcoords='offset points', ha='left',
arrowprops=dict(arrowstyle="->"))
fig.suptitle('CPD - Boston 2018', fontsize=20)
plt.ylabel('CPD', fontsize=18)
#Plot lat long
from pylab import rcParams
data3.Lat.replace(-1, None, inplace=True)
data3.Long.replace(-1, None, inplace=True)
rcParams["figure.figsize"] = 21,11
plt.subplots(figsize=(11,6))
sns.scatterplot(x='Lat',
y='Long',
hue='DISTRICT',
alpha=0.1,
data=data3)
plt.legend(loc=2)
