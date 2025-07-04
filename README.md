# cintel-02-data
P2 Browser Interactive Data App & Engage

# Install packages
import numpy as np
import matplotlib.pyplot as plt
import plotly.express as px
from palmerpenguins import load_penguins
import seaborn as sns

from shiny.express import ui, render, input
from shinywidgets import render_widget, render_plotly            

# Create Page title
ui.page_opts(title="Brenda's Penguin Data", fillable=True)

# Create sidebar with a 2nd level header, a horizontal ruler & a GitHub hyperlink
# sidebar
with ui.sidebar(open="open"):
    ui.h2("Palmer Penguins Sidebar")     #2nd level header
    ui.hr()                              #horizontal ruler

    ui.a(
        "Brenda's GitHub Repo",                # Github hyperlink
        href="https://github.com/bfuemmeler/cintel-02-data",
        target="_blank",
    )

# Controls
ui.h3("Controls")

# Add Dropdown
# add dropdown
    ui.input_selectize(
        "selected_attribute",
        "Choose attribute",
        ["bill_length_mm", "bill_depth_mm", "flipper_length_mm", "body_mass_g"],
    )

# Input Numeric
# input numeric
    ui.input_numeric(
        "plotly_bin_count",
        "Number of Plotly bins",
        30,
        min=5,
        max=100,
    )

# Input a Slider
# slider for Seaborn bins
    ui.input_slider(
        "n_bins",
        "Number of Seaborn bins",
        1,
        100,
        30, 
        step=1,
    )

# Input a Checkbox group
# checkbox group
    ui.input_checkbox_group(
        "selected_species",
        "Species in table",
        ["Adelie", "Gentoo", "Chinstrap"],
        selected=["Adelie"],
        inline=True,
    )
    
# Use "with" columns
with ui.layout_columns():

# Build a Seaborn Histogram
with ui.layout_columns():
    @render.plot(
        alt="A Seaborn histogram on penguin body mass in grams."
    )
    def seaborn_hist():                       
        ax = sns.histplot(
            data=penguins,
            x="body_mass_g",
            bins=input.n_bins()               
        )
        ax.set(title="Palmer Penguins",
               xlabel="Mass (g)",
               ylabel="Count")
        return ax

# Build a Plotly Histogram
with ui.layout_columns():
    @render_widget
    def plotly_hist():                       
        fig = px.histogram(
            penguins,
            x="body_mass_g",
            nbins=input.plotly_bin_count()    
        ).update_layout(
            title=dict(text="Penguin Mass", x=0.5),
            yaxis_title="Count",
            xaxis_title="Body Mass (g)",
        )
        return fig

# Build a Data Table
#DataTable
with ui.layout_columns():       
    @render.data_frame                         
    def penguins_df():
        return render.DataTable(penguins)  

# Build a Data Grid
#DataGrid
with ui.layout_columns():
    @render.DataGrid                           
    def penguin_table():
        return render.DataGrid(penguins)

# Build a Plotly Scatterplot
with ui.card(full_screen=True):
    ui.card_header("Plotly Scatterplot: Species")
    
    @render_plotly
    def plotly_scatterplot():
        fig = px.scatter(
            data_frame=penguins,              # 1) the DataFrame
            x=input.selected_attribute(),     # 2) x‑axis (chosen in sidebar)
            y="flipper_length_mm",            # 3) y‑axis
            color="species",                  # 4) colour legend
            symbol="species",                 # 5) different markers per species
            title=f"{input.selected_attribute()} vs Flipper length"  # 6) title
        ).update_layout(
            xaxis_title=input.selected_attribute().replace("_", " ").title(),
            yaxis_title="Flipper length (mm)"
        )
        return fig
        
