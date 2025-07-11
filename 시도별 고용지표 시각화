import streamlit as st
import pandas as pd
import folium
from streamlit_folium import st_folium
import plotly.express as px
import json
import warnings
warnings.filterwarnings("ignore")
st.set_page_config(layout="wide")
st.title("지역별 고용지표 대시보드")
uploaded_file = st.file_uploader("엑셀 파일 업로드", type=["xlsx"])
if uploaded_file:
    data = pd.read_excel(uploaded_file, skiprows=1, header=1)
    data.columns = data.columns.str.lower()
    data = data.dropna(subset=['ednum'])
    data = data.drop_duplicates(subset=['ednum']) 
cols = [data.columns[3]] + list(data.columns[20:])
month = data[cols]
month_view = pd.DataFrame()
sido_dict={11:"서울",21:"부산",22:"대구",23:"인천",24:"광주",25:"대전",
          26:"울산",29:"세종",31:"경기",32:"강원",33:"충북",34:"충남",
          35:"전북",36:"전남",37:"경북",38:"경남",39:"제주"}
month_view["지역명"]=month["ednum"].map(sido_dict)
month_view["경제활동인구"]=month["실업자계"]+month["취업자계"]
month_view["15세이상인구"]=month["대상자계"]
month_view["경제활동참가율(%)"]=month_view["경제활동인구"]/month_view["15세이상인구"]*100
month_view["고용률(%)"]=month["취업자계"]/month_view["15세이상인구"]*100
month_view["실업률(%)"]=month["실업자계"]/month_view["경제활동인구"]*100
cols2 = [month_view.columns[0]] + list(month_view.columns[3:])
month_view=month_view[cols2]
st.subheader("고용지표 표")
st.dataframe(month_view, use_container_width=True)
indicator = st.selectbox("시각화할 지표를 선택하세요", ["고용률(%)", "실업률(%)", "경제활동참가율(%)"])
df = month_view.copy()
st.subheader(f"시도별 {indicator} 비교")
fig = px.bar(
    df,
    x="지역명",
    y=indicator,
    color=indicator,
    color_continuous_scale="YlGnBu",
    text=df[indicator].map(lambda x: f"{x:.2f}")
)
st.plotly_chart(fig, use_container_width=True)
import folium
from streamlit_folium import st_folium
sido_coords = {
    "서울": [37.5665, 126.9780], "부산": [35.1796, 129.0756],
    "대구": [35.8714, 128.6014], "인천": [37.4563, 126.7052],
    "광주": [35.1595, 126.8526], "대전": [36.3504, 127.3845],
    "울산": [35.5384, 129.3114], "세종": [36.4800, 127.2890],
    "경기": [37.4138, 127.5183], "강원": [37.8228, 128.1555],
    "충북": [36.6357, 127.4917], "충남": [36.5184, 126.8000],
    "전북": [35.7167, 127.1440], "전남": [34.8161, 126.4630],
    "경북": [36.5755, 128.5056], "경남": [35.4606, 128.2132],
    "제주": [33.4996, 126.5312]
}
map_indicator = st.selectbox("지도에 표시할 지표를 선택하세요", ["고용률(%)", "실업률(%)", "경제활동참가율(%)"])
month_view["위도"] = month_view["지역명"].map(lambda x: sido_coords.get(x, [None, None])[0])
month_view["경도"] = month_view["지역명"].map(lambda x: sido_coords.get(x, [None, None])[1])
m = folium.Map(location=[36.5, 127.8], zoom_start=7)
for i, row in month_view.iterrows():
    lat, lon = row["위도"], row["경도"]
    name = row["지역명"]
    value = row[map_indicator]
    if pd.isna(lat) or pd.isna(lon) or pd.isna(value):
        continue
    if map_indicator == "고용률(%)":
        if value >= 70:
            color = "green"
        elif value >= 60:
            color = "orange"
        else:
            color = "red"
    elif map_indicator == "실업률(%)":
        if value >= 5:
            color = "red"
        elif value >= 3:
            color = "orange"
        else:
            color = "green"
    elif map_indicator == "경제활동참가율(%)":
        if value >= 70:
            color = "green"
        elif value >= 60:
            color = "orange"
        else:
            color = "red"
    folium.CircleMarker(
        location=[lat, lon],
        radius=value / 3,  
        color=color,
        fill=True,
        fill_opacity=0.7,
        popup=f"{name}: {value:.2f}%",
        tooltip=f"{name} ({value:.2f}%)"
    ).add_to(m)
st.subheader(f"시도별 {map_indicator} 지도 시각화")
st_data = st_folium(m, width=725)
