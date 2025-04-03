import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
from io import BytesIO

st.set_page_config(page_title="Cap Table Tool", layout="centered")
st.title("🧾 Cap Table & Dilution Tool")

st.markdown("Manage and visualize your startup's ownership structure with support for share classes and funding rounds.")

st.sidebar.header("Settings")
show_pie_chart = st.sidebar.checkbox("Show Pie Chart", value=True)
include_options_pool = st.sidebar.checkbox("Include Options Pool", value=False)

# Section 1: Shareholders
st.subheader("🧍 Current Shareholders")

with st.form("cap_table_form"):
    num_shareholders = st.number_input("Number of shareholders", min_value=1, max_value=20, value=3)
    shareholder_data = []

    for i in range(int(num_shareholders)):
        name = st.text_input(f"Shareholder {i+1} Name", key=f"name_{i}")
        shares = st.number_input(f"Shares owned", min_value=0, key=f"shares_{i}")
        share_class = st.selectbox("Share Class", ["Ordinary", "Preferred"], key=f"class_{i}")
        shareholder_data.append((name, shares, share_class))

    if include_options_pool:
        options_pool = st.number_input("Options Pool Shares", min_value=0, key="options_pool")
        shareholder_data.append(("Options Pool", options_pool, "Ordinary"))

    submitted = st.form_submit_button("Generate Cap Table")

if submitted:
    df = pd.DataFrame(shareholder_data, columns=["Shareholder", "Shares", "Share Class"])
    df = df[df["Shares"] > 0]
    total_shares = df["Shares"].sum()
    df["Ownership %"] = round((df["Shares"] / total_shares) * 100, 2)

    st.subheader("📊 Cap Table (Current)")
    st.dataframe(df, use_container_width=True)
    st.markdown(f"**Total Shares:** {int(total_shares)}")

    # Section 2: Pre/Post-Money Valuation
    st.subheader("💰 Pre/Post-Money Valuation (Optional)")
    pre_money = st.number_input("Pre-Money Valuation (£)", min_value=0, value=5000000, step=100000)
    investment_amount = st.number_input("New Investment Amount (£)", min_value=0, step=10000)

    if investment_amount > 0 and pre_money > 0:
        new_shares = round((investment_amount / pre_money) * total_shares)
        investor_ownership = round((new_shares / (total_shares + new_shares)) * 100, 2)
        post_money = pre_money + investment_amount
        total_after = total_shares + new_shares

        df["Post-Investment %"] = round((df["Shares"] / total_after) * 100, 2)

        # Add new investor row
        investor_row = pd.DataFrame([["New Investor", new_shares, "Preferred", None, investor_ownership]],
                                    columns=["Shareholder", "Shares", "Share Class", "Ownership %", "Post-Investment %"])
        post_df = pd.concat([df, investor_row], ignore_index=True)

        st.subheader("📉 Post-Investment Cap Table")
        st.dataframe(post_df.drop(columns="Ownership %"), use_container_width=True)

        st.markdown(f"""
        **Post-Money Valuation:** £{post_money:,.0f}  
        **New Shares Issued:** {new_shares}  
        **New Total Shares:** {total_after}  
        **Investor Ownership:** {investor_ownership}%
        """)

        if show_pie_chart:
            fig, ax = plt.subplots()
            post_df.plot.pie(
                y="Post-Investment %",
                labels=post_df["Shareholder"],
                autopct='%1.1f%%',
                legend=False,
                figsize=(6, 6),
                ax=ax
            )
            st.pyplot(fig)

    # Section 3: Export Excel
    def convert_df_to_excel(df1, df2=None):
        output = BytesIO()
        with pd.ExcelWriter(output, engine='xlsxwriter') as writer:
            df1.to_excel(writer, index=False, sheet_name='Pre-Investment')
            if df2 is not None:
                df2.to_excel(writer, index=False, sheet_name='Post-Investment')
        output.seek(0)
        return output

    st.subheader("📥 Export Cap Table")
    excel_data = convert_df_to_excel(df, post_df if investment_amount > 0 else None)
    st.download_button("Download Excel File", excel_data, file_name="cap_table_with_share_classes.xlsx")
