import streamlit as st
import sqlite3
from datetime import datetime

# Page Configuration
st.set_page_config(page_title="Local Store Inventory Tracker", page_icon="📍", layout="centered")

# Initialize SQLite Database
conn = sqlite3.connect("inventory.db", check_same_thread=False)
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS inventory (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        store_name TEXT,
        product_name TEXT,
        price TEXT,
        status TEXT,
        last_updated TEXT
    )
""")
conn.commit()

# Populate sample data if table is empty
cursor.execute("SELECT COUNT(*) FROM inventory")
if cursor.fetchone()[0] == 0:
    sample_data = [
        ("Downtown Electronics", "USB-C Fast Charger", "$15", "In Stock", "Today, 09:00 AM"),
        ("Downtown Electronics", "Wireless Earbuds", "$35", "Out of Stock", "Today, 10:30 AM"),
        ("Metro Footwear", "Leather Boots Size 42", "$60", "Low Stock", "Today, 08:15 AM")
    ]
    cursor.executemany("INSERT INTO inventory (store_name, product_name, price, status, last_updated) VALUES (?, ?, ?, ?, ?)", sample_data)
    conn.commit()

st.title("📍 Local Store Inventory Tracker")
st.markdown("Check if items are available nearby before you waste money on travel.")

# Navigation Tabs
tab1, tab2 = st.tabs(["🔍 Search Products (Customer)", "⚙️ Update Stock (Store Owner)"])

# TAB 1: CUSTOMER VIEW
with tab1:
    st.subheader("Live Shelf Availability")
    search_query = st.text_input("Search for a product or store name...")
    
    if search_query:
        cursor.execute("SELECT store_name, product_name, price, status, last_updated FROM inventory WHERE product_name LIKE ? OR store_name LIKE ?", 
                       (f"%{search_query}%", f"%{search_query}%"))
    else:
        cursor.execute("SELECT store_name, product_name, price, status, last_updated FROM inventory")
        
    results = cursor.fetchall()
    
    if not results:
        st.warning("No products found matching your search.")
    
    for row in results:
        store, product, price, status, updated = row
        
        # Color-code stock status
        if status == "In Stock":
            status_color = "green"
        elif "Low" in status:
            status_color = "orange"
        else:
            status_color = "red"
            
        st.markdown(f"### {product} — **{price}**")
        st.markdown(f"**Store:** {store}")
        st.markdown(f"**Status:** :{status_color}[**{status}**]")
        st.caption(f"Last confirmed: {updated}")
        st.divider()

# TAB 2: STORE OWNER VIEW
with tab2:
    st.subheader("Store Owner Dashboard")
    st.markdown("Update your store inventory live so customers know what is on the shelf.")
    
    cursor.execute("SELECT id, store_name, product_name, status FROM inventory")
    items = cursor.fetchall()
    
    if items:
        selected_item = st.selectbox("Select product to update:", items, format_func=lambda x: f"{x[1]} - {x[2]} (Currently: {x[3]})")
        new_status = st.radio("Set New Status:", ["In Stock", "Low Stock", "Out of Stock"])
        
        if st.button("Update Status Now"):
            now_str = datetime.now().strftime("%b %d, %I:%M %p")
            cursor.execute("UPDATE inventory SET status = ?, last_updated = ? WHERE id = ?", (new_status, now_str, selected_item[0]))
            conn.commit()
            st.success(f"Successfully updated status to '{new_status}'!")
            st.rerun()
