# AI Agent Instructions for Performance Analysis Dashboard

## Project Overview
This is a Streamlit-based performance analytics dashboard with multiple pages and components. The application provides visualization and analysis of metrics including traffic, user engagement, healthcare data, and more.

## Application Structure
- `app.py` - Main application entry point with core dashboard layout
- `pages/` - Directory containing individual page components:
  - `Home.py` - Landing page
  - `Messages.py` - Communication interface
  - `Favorites.py` - Saved items/reports
  - `Health_Care.py` - Healthcare metrics
  - `Map.py` - Location-based analytics
  - `Reports.py` - Detailed analytics reports
  - `Settings.py` - Application configuration

## Key Architectural Patterns
1. **Page Navigation**
   - Uses Streamlit's page system with sidebar navigation
   - Example pattern from `app.py`:
   ```python
   st.sidebar.page_link("/pages/Home.py", label="Home", icon="🏠")
   ```

2. **Layout Structure**
   - Pages use column-based layouts for responsive design
   - Common pattern:
   ```python
   col1, col2 = st.columns(2)
   with col1:
       st.subheader("Section 1")
   with col2:
       st.subheader("Section 2")
   ```

3. **Data Visualization**
   - Uses Streamlit's built-in charting (`st.line_chart`, `st.bar_chart`)
   - Metrics displayed using `st.metric` with delta values

## Development Workflow
1. **Running the Application**
   ```powershell
   streamlit run app.py
   ```

2. **Adding New Pages**
   - Create new `.py` file in `pages/` directory
   - Add corresponding sidebar link in `app.py`

## Dependencies
Main libraries:
- streamlit
- numpy
- pandas
- plotly (for advanced visualizations)

## Best Practices
1. **Page Organization**
   - Keep page files focused on single responsibility
   - Use consistent layout patterns across pages
   - Maintain sidebar navigation structure in `app.py`

2. **Data Handling**
   - Use pandas DataFrames for structured data
   - Implement proper data loading and caching when needed
   - Current sample data uses random generation - replace with actual data sources

3. **UI Components**
   - Follow established column layout patterns
   - Use consistent icon styles (emoji-based)
   - Maintain standard metric display format

---
> Source: [secureterminal/performance-analysis](https://github.com/secureterminal/performance-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-26 -->
