# main.py-import pandas as pd
import plotly.express as px
import streamlit as st

st.set_page_config(page_title="영화 데이터 그래프 도감 2 - 분포와 관계", layout="wide")
st.title("영화 데이터 그래프 도감 2 - 분포와 관계")

DATA_URL = (
    "https://raw.githubusercontent.com/greatsong/modudata/main/data/kobis_movies.csv"
)


@st.cache_data
def load_data():
    # 1년간 박스오피스 10위권에 든 영화 216편의 요약표를 불러옵니다
    df = pd.read_csv(DATA_URL)
    # 장르가 세로막대 기호(|)로 여러 개 적힌 영화는 첫 번째 장르만 씁니다
    if "genre" in df.columns:
        df["장르"] = df["genre"].str.split("|").str[0]
    return df


df = load_data()

# 컬럼명 유연성 확보 (데이터셋 컬럼명 대응)
audi_col = (
    "audiAcc"
    if "audiAcc" in df.columns
    else ("관객수" if "관객수" in df.columns else None)
)
scrn_col = (
    "scrnCnt"
    if "scrnCnt" in df.columns
    else ("스크린수" if "스크린수" in df.columns else None)
)
title_col = (
    "movieNm"
    if "movieNm" in df.columns
    else ("영화명" if "영화명" in df.columns else None)
)

# ── 그래프 1. 장르별 영화 편수 도넛 ──
st.header("1. 장르별 영화 편수 (도넛)")
genre_count = df["장르"].value_counts().reset_index()
genre_count.columns = ["장르", "편수"]

fig1 = px.pie(
    genre_count,
    names="장르",
    values="편수",
    hole=0.45,  # 가운데 구멍을 뚫어 도넛 모양으로
)
fig1.update_traces(hovertemplate="%{label}<br>%{value}편 (%{percent})<extra></extra>")
st.plotly_chart(fig1, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note1")

st.divider()

# ── 그래프 2. 장르별 관객수 분포 (상자 그림 / Box Plot) ──
st.header("2. 장르별 관객수 분포 (상자 그림)")
st.caption(
    "각 장르 내 영화들의 관객수 중앙값, 사분위수 및 아웃라이어(대흥행작)를 확인합니다."
)

if audi_col:
    fig2 = px.box(
        df,
        x="장르",
        y=audi_col,
        color="장르",
        points="all",  # 실제 개별 영화 점도 함께 표시
        hover_name=title_col,
        labels={audi_col: "누적 관객수(명)", "장르": "영화 장르"},
    )
    fig2.update_layout(showlegend=False)
    st.plotly_chart(fig2, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note2")

st.divider()

# ── 그래프 3. 스크린수와 관객수의 관계 (산점도 / Scatter Plot) ──
st.header("3. 스크린수와 관객수의 관계 (산점도)")
st.caption(
    "개봉 당시 확보한 스크린수가 많을수록 누적 관객수도 늘어나는지 양의 상관관계를 살펴봅니다."
)

if scrn_col and audi_col:
    fig3 = px.scatter(
        df,
        x=scrn_col,
        y=audi_col,
        color="장르",
        hover_name=title_col,
        labels={scrn_col: "스크린수(개)", audi_col: "누적 관객수(명)"},
        trendline="ols",  # 전체 경향성을 보여주는 추세선 추가
    )
    st.plotly_chart(fig3, use_container_width=True)

st.text_input("이 그래프로 알 수 있는 것", key="note3")
