# Chart Rendering Specifications

> Python matplotlib code patterns for each chart type used in the SEO Monthly Report.
> All charts are exported as PNG (300 DPI, transparent or white background).

---

## Global Chart Style

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import numpy as np

# Apply to all charts
plt.rcParams.update({
    'font.family': 'sans-serif',
    'font.sans-serif': ['Calibri', 'Arial', 'DejaVu Sans'],
    'font.size': 11,
    'axes.facecolor': 'white',
    'figure.facecolor': 'white',
    'axes.grid': True,
    'grid.alpha': 0.3,
    'grid.axis': 'y',
    'axes.spines.top': False,
    'axes.spines.right': False,
})

# Color constants
BLUE = '#4472C4'
RED = '#FF0000'
GREY_LIGHT = '#B7B7B7'
GREY_DARK = '#2D2D2D'
ORANGE = '#FFD666'
RED_LINE = '#E74C3C'
GREEN = '#27AE60'
```

## Number Formatting Helper

```python
def format_value(val):
    """Format numbers as K/M suffix"""
    if val >= 1_000_000:
        return f'{val/1_000_000:.1f}M'
    elif val >= 1_000:
        return f'{val/1_000:.1f}K'
    else:
        return str(int(val))
```

---

## Type A: Grouped Bar Chart — 2 Series (Slides 4, 5, 6)

Used for: Brand volume, Brand+Category volume, Non-brand volume (YoY comparison)

```python
def chart_grouped_bar_2series(months, prev_year_data, curr_year_data, 
                               prev_year_label, curr_year_label, title,
                               output_path):
    fig, ax = plt.subplots(figsize=(14, 6))
    
    x = np.arange(len(months))
    width = 0.35
    
    bars1 = ax.bar(x - width/2, prev_year_data, width, label=prev_year_label, 
                   color=BLUE, zorder=3)
    
    # Current year: only plot non-None values
    curr_clean = [v if v is not None else 0 for v in curr_year_data]
    curr_colors = [RED if v is not None else 'none' for v in curr_year_data]
    bars2 = ax.bar(x + width/2, curr_clean, width, label=curr_year_label,
                   color=curr_colors, zorder=3)
    
    # Data labels
    for bar, val in zip(bars1, prev_year_data):
        if val > 0:
            ax.text(bar.get_x() + bar.get_width()/2., bar.get_height() + 200,
                    format_value(val), ha='center', va='bottom', fontsize=8, color=BLUE)
    
    for bar, val in zip(bars2, curr_year_data):
        if val is not None and val > 0:
            ax.text(bar.get_x() + bar.get_width()/2., bar.get_height() + 200,
                    format_value(val), ha='center', va='bottom', fontsize=8, color=RED)
    
    ax.set_xlabel('')
    ax.set_xticks(x)
    ax.set_xticklabels(months)
    ax.set_title(title, fontsize=14, fontweight='bold', pad=15)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda v, p: format_value(v)))
    ax.legend(loc='lower center', bbox_to_anchor=(0.5, -0.15), ncol=2, frameon=False)
    
    plt.tight_layout()
    fig.savefig(output_path, dpi=300, bbox_inches='tight', facecolor='white')
    plt.close()
```

---

## Type B: Grouped Bar Chart — 3 Series (Slide 14)

Used for: GSC Impression/Click Trend (3-year comparison)

```python
def chart_grouped_bar_3series(months, year1_data, year2_data, year3_data,
                               year1_label, year2_label, year3_label, title,
                               output_path):
    fig, ax = plt.subplots(figsize=(14, 5))
    
    x = np.arange(len(months))
    width = 0.25
    
    ax.bar(x - width, year1_data, width, label=year1_label, color=GREY_LIGHT, zorder=3)
    ax.bar(x, year2_data, width, label=year2_label, color=GREY_DARK, zorder=3)
    
    # Year 3: only non-None values
    y3_clean = [v if v is not None else 0 for v in year3_data]
    y3_colors = [ORANGE if v is not None else 'none' for v in year3_data]
    ax.bar(x + width, y3_clean, width, label=year3_label, color=y3_colors, zorder=3)
    
    # Data labels for year2 and year3
    for i, val in enumerate(year2_data):
        if val > 0:
            ax.text(i, val + val*0.02, format_value(val), ha='center', 
                    va='bottom', fontsize=7, color=GREY_DARK)
    for i, val in enumerate(year3_data):
        if val is not None and val > 0:
            ax.text(i + width, val + val*0.02, format_value(val), ha='center',
                    va='bottom', fontsize=7, color='#B8860B')
    
    ax.set_xticks(x)
    ax.set_xticklabels(months)
    ax.set_title(title, fontsize=13, fontweight='bold', pad=12)
    ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda v, p: format_value(v)))
    ax.legend(loc='lower center', bbox_to_anchor=(0.5, -0.18), ncol=3, frameon=False)
    
    plt.tight_layout()
    fig.savefig(output_path, dpi=300, bbox_inches='tight', facecolor='white')
    plt.close()
```

---

## Type C: Combo Chart — Stacked Bar + Line (Slide 11)

Used for: GA4 Organic Detail (Revenue + Sessions bars, Transactions line)

```python
def chart_combo_bar_line(months, revenue_data, session_data, transaction_data,
                          output_path):
    fig, ax1 = plt.subplots(figsize=(14, 6))
    ax2 = ax1.twinx()
    
    x = np.arange(len(months))
    width = 0.6
    
    # Stacked bars
    bars_rev = ax1.bar(x, revenue_data, width, label='Organic Revenue', 
                       color=GREY_LIGHT, zorder=3)
    bars_ses = ax1.bar(x, session_data, width, bottom=revenue_data,
                       label='Organic Session', color=ORANGE, zorder=3)
    
    # Line for transactions
    line = ax2.plot(x, transaction_data, color=RED_LINE, marker='o', 
                    linewidth=2, markersize=6, label='Organic Transaction', zorder=4)
    
    # Data labels: sessions on bars
    for i, (rev, ses) in enumerate(zip(revenue_data, session_data)):
        ax1.text(i, rev + ses + (rev+ses)*0.02, format_value(ses),
                ha='center', va='bottom', fontsize=7, color='#B8860B')
    
    # Data labels: transactions on line
    for i, trx in enumerate(transaction_data):
        ax2.text(i, trx + 2, str(int(trx)), ha='center', va='bottom',
                fontsize=8, fontweight='bold', color=RED_LINE)
    
    ax1.set_xticks(x)
    ax1.set_xticklabels(months, fontsize=9)
    ax1.yaxis.set_major_formatter(mticker.FuncFormatter(lambda v, p: format_value(v)))
    ax2.set_ylim(0, max(transaction_data) * 1.3)
    
    # Combined legend
    lines1, labels1 = ax1.get_legend_handles_labels()
    lines2, labels2 = ax2.get_legend_handles_labels()
    ax1.legend(lines1 + lines2, labels1 + labels2,
              loc='lower center', bbox_to_anchor=(0.5, -0.15), ncol=3, frameon=False)
    
    plt.tight_layout()
    fig.savefig(output_path, dpi=300, bbox_inches='tight', facecolor='white')
    plt.close()
```

---

## Type D: Pie Chart — Side by Side (Slide 16)

Used for: Share of Clicks & AI Search Share of Voice

```python
def chart_dual_pie(soc_data, sov_data, output_path):
    """
    soc_data: dict {site_name: percentage}
    sov_data: dict {site_name: percentage}
    """
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 7))
    
    # Color palette for sites
    colors = ['#FF7B52', '#4472C4', '#FFD666', '#E74C3C', '#27AE60',
              '#9B59B6', '#3498DB', '#E67E22', '#1ABC9C', '#95A5A6',
              '#D35400', '#8E44AD', '#2ECC71']
    
    def make_pie(ax, data, title_text, title_icon):
        labels = list(data.keys())
        sizes = list(data.values())
        
        # Only label slices > 5%
        def autopct_func(pct):
            return f'{pct:.1f}%' if pct > 5 else ''
        
        wedges, texts, autotexts = ax.pie(
            sizes, labels=None, autopct=autopct_func,
            colors=colors[:len(labels)], startangle=90,
            pctdistance=0.75, textprops={'fontsize': 9}
        )
        
        # Add labels for large slices outside
        for i, (wedge, label, size) in enumerate(zip(wedges, labels, sizes)):
            if size > 5:
                ang = (wedge.theta2 - wedge.theta1)/2. + wedge.theta1
                y = np.sin(np.deg2rad(ang))
                x = np.cos(np.deg2rad(ang))
                ax.annotate(f'{label}: {size}%',
                           xy=(x*0.7, y*0.7), fontsize=8, fontweight='bold',
                           ha='left' if x > 0 else 'right')
        
        ax.set_title(f'{title_icon} {title_text}', fontsize=13, fontweight='bold', pad=15)
    
    make_pie(ax1, soc_data, 'Google SoC', '🔍')
    make_pie(ax2, sov_data, 'AI Search SoV', '🌐')
    
    # Shared legend at bottom
    all_labels = list(soc_data.keys())
    legend_handles = [plt.Rectangle((0,0),1,1, fc=colors[i]) for i in range(len(all_labels))]
    fig.legend(legend_handles, all_labels, loc='lower center', 
              ncol=min(6, len(all_labels)), fontsize=8, frameon=False,
              bbox_to_anchor=(0.5, 0.02))
    
    plt.tight_layout(rect=[0, 0.08, 1, 1])
    fig.savefig(output_path, dpi=300, bbox_inches='tight', facecolor='white')
    plt.close()
```

---

## Export Settings

All charts:
- **DPI:** 300
- **Format:** PNG
- **Background:** White (`facecolor='white'`)
- **Tight layout:** Always use `bbox_inches='tight'`
- **File naming:** `chart_slide{slide_number}_{description}.png`
  - e.g., `chart_slide04_brand_volume.png`
  - e.g., `chart_slide14_gsc_impression_trend.png`
