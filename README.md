import streamlit as st
import pandas as pd
import plotly.express as px

# 1. Dashboard Layout Settings
st.set_page_config(page_title="DataCo Supply Chain Analytics", layout="wide")
st.title("📦 DataCo Smart Supply Chain Performance Live Dashboard")
st.markdown("An interactive analytics portfolio project evaluating shipping efficiency and product profitability.")

# 2. Optimized Cloud Ingestion Data Pipeline
@st.cache_data
def load_data():
    url = "https://githubusercontent.com"
    df = pd.read_csv(url, encoding="latin1")
    df.columns = [c.replace(' ', '_') for c in df.columns]
    return df

try:
    df = load_data()
    
    # 3. High-Level Global Metric Operational KPIs
    col1, col2, col3 = st.columns(3)
    col1.metric("Total Transactions", f"{len(df):,}")
    col2.metric("Total Revenue Generated", f"${df['Sales'].sum():,.2f}")
    col3.metric("Late Delivery Risk Rate", f"{(df['Delivery_Status'] == 'Late delivery').mean() * 100:.2f}%")

    st.markdown("---")

    # 4. Interactive Sidebar Component Filter (Filer Setup)
    st.sidebar.header("Filter Dashboard View")
    selected_segment = st.sidebar.multiselect(
        "Select Customer Segment:", 
        options=list(df['Customer_Segment'].unique()), 
        default=list(df['Customer_Segment'].unique())
    )
    
    # Apply backend filter slicing logic to dataframe
    filtered_df = df[df['Customer_Segment'].isin(selected_segment)]

    # 5. Comparative Performance Interactive Visualizations
    chart_col1, chart_col2 = st.columns(2)

    with chart_col1:
        st.subheader("🛒 Profitability by Product Category")
        category_profit = filtered_df.groupby('Category_Name')['Order_Profit_Per_Order'].sum().reset_index().sort_values(by='Order_Profit_Per_Order', ascending=False).head(10)
        fig_profit = px.bar(category_profit, x='Order_Profit_Per_Order', y='Category_Name', orientation='h', color='Order_Profit_Per_Order', color_continuous_scale='Viridis')
        st.plotly_chart(fig_profit, use_container_width=True)

    with chart_col2:
        st.subheader("🚚 Shipping Mode vs Delivery Delays")
        shipping_delay = filtered_df.groupby(['Shipping_Mode', 'Delivery_Status']).size().reset_index(name='Count')
        fig_shipping = px.bar(shipping_delay, x='Shipping_Mode', y='Count', color='Delivery_Status', barmode='group')
        st.plotly_chart(fig_shipping, use_container_width=True)

except Exception as e:
    st.error(f"⚠️ Error loading data from cloud: {e}")
