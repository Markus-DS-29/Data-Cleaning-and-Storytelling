# Data Cleaning & Storytelling with Python
## Case Study: Help an e-commerce company  by performing Data Analysis to settle an ongoing debate: whether or not it’s beneficial to discount products.
## Tasks
  1. assess the data quality of the dataset 
  2. take on many data cleaning tasks to make given data usable and trustable
  3. prepare data visualisations using Pandas, Seaborn & Matplotlib
  4. identify, analyse, interpret and present relevant data
  5. prepare conclusions and recommendations

## Examples
### Creating Categories

1. Identifying Keywords in product title and description

```   
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from collections import Counter

nltk.download('punkt')
nltk.download('stopwords')

def extract_keywords(text):
    stop_words = set(stopwords.words('english'))
    words = word_tokenize(text.lower())
    keywords = [word for word in words if word.isalpha() and word not in stop_words]
    return keywords
product_category_df['keywords'] = product_category_df['desc'].apply(extract_keywords)
all_keywords = sum(product_category_df['keywords'], [])
keyword_counts = Counter(all_keywords)

# Display the most common keywords
keywords_list_desc = keyword_counts.most_common(30)

```
2. Set categories using keywords, type and assign 'other' to the remainung products 

```
product_category_df.loc[product_category_df["desc"].str.contains("keyboard|keypad", case=False), "category"] = ", perepheric_devices"
product_category_df.loc[product_category_df["name"].str.contains("iphone", case=False) & (product_category_df["sku"].str.contains("app", case=False)), "category"] = ", iphone"
product_category_df.loc[product_category_df["name"].str.contains("ipod|watch", case=False) & (product_category_df["sku"].str.contains("app", case=False)) &(product_category_df["category"] != "iphone"), "category"] = ", ipod/iwatch"
product_category_df.loc[product_category_df["name"].str.contains("ipad|tablet", case=False) & (product_category_df["sku"].str.contains("app", case=False)) &  (product_category_df["category"] != "iphone | ipod/iwatch"), "category"] = ", tablet"
product_category_df.loc[(product_category_df["desc"].str.contains("apple", case=False)) & (~product_category_df["sku"].str.contains("app", case=False)), "category"] = ", apple_generica"
product_category_df.loc[(product_category_df["name"].str.contains("case|cover", case=False)) & (~product_category_df["sku"].str.contains("app", case=False)), "category"] = ", cases"
product_category_df.loc[product_category_df["name"].str.contains("drive|disk|hdd|ssd|memory|thunderbolt", case=False), "category"] = ", memory"
product_category_df.loc[product_category_df["name"].str.contains("monitor|hdmi", case=False), "category"] = ", monitor"
product_category_df.loc[product_category_df["name"].str.contains("camera|gopro", case=False), "category"] = ", perepheric_devices"
product_category_df.loc[product_category_df["type"].str.contains("5398"), "category"] = ", perepheric_devices"
product_category_df.loc[product_category_df["type"].str.contains("13005399"), "category"] = ", perepheric_devices"
product_category_df.loc[product_category_df["type"].str.contains("12175397"), "category"] = ", server"
product_category_df.loc[(product_category_df["category"] == "") & (product_category_df["sku"].str.contains("app", case=False)), "category"] = "apple_divers"

product_category_df.loc[(product_category_df["category"] == ""), "category"] = "other"

```
3. Check the relevance of the categories based on the respective revenues

```
import matplotlib.pyplot as plt
import seaborn as sns

# Aggregating and sorting the data
aggregated_data = product_category_df.groupby('category').agg({
    'sku': 'count',
    'product_quantity': 'sum',
    'product_price': 'mean',
    'discount_price': 'mean',
    'discount_perc': 'mean',
    'revenue': 'sum',
    'perc_total_rev': 'sum'
}).sort_values('perc_total_rev', ascending=False)

# Reset index to have 'category' as a column
aggregated_data = aggregated_data.reset_index()

# Create a bar plot
plt.figure(figsize=(6, 4))
barplot = sns.barplot(data=aggregated_data, x='revenue', y='category')

# Adding titles and labels
plt.title('Revenue by Category')
plt.xlabel('Revenue im Million EUR')

# Add text labels for 'perc_total_rev'
for index, row in aggregated_data.iterrows():
    barplot.text(row['revenue'], index, f"{row['perc_total_rev']:.2f}%", color='black', ha="left", va="center")

# Remove the frame
barplot.spines['top'].set_visible(False)
barplot.spines['right'].set_visible(False)
barplot.spines['left'].set_visible(False)
barplot.spines['bottom'].set_visible(False)

# Show the plot
plt.show()

```
<img src="https://raw.githubusercontent.com/Markus-DS-29/Data-Cleaning-and-Storytelling/main/Revenue%20by%20Category.png" alt="Revenue by Category" style="width: 616px; height: auto;">

### Analyse Revenues

```
#show revenue and discount_perc in one plot

# Category: Memory
# Resample data

memory_mean = grouped_categories_disc_perc.resample('W', on='date')['memory'].mean()
memory_revenue = grouped_categories_rev.resample('W', on='date')['memory'].sum()

# Plotting
fig, ax1 = plt.subplots(figsize=(10, 4))

# Plot the revenue with the primary y-axis
ax1.plot(memory_revenue, color='b', label='Revenue')
ax1.set_ylabel('Revenue', color='b')
ax1.tick_params(axis='y', labelcolor='b')
ax1.set_ylim(0, 100000)  # Set the y-axis limit for revenue

# Create a second y-axis for the discount percentage
ax2 = ax1.twinx()
ax2.plot(memory_mean, color='r', label='Discount Percentage (Mean)')
ax2.set_xlabel('Date')
ax2.set_ylabel('Discount Percentage (Mean)', color='r')
ax2.tick_params(axis='y', labelcolor='r')
ax2.set_ylim(0, 0.4)

# Adding a title and a grid
plt.title('Memory Sales (32% of total revenues): Weekly Revenue and Discount Percentage (Mean)')
#ax1.grid()

# Adding legends
ax1_legend = ax1.legend(loc='upper left', bbox_to_anchor=(0.05, 1))
ax2_legend = ax2.legend(loc='upper left', bbox_to_anchor=(0.05, 0.9))

# Show the plot
plt.show()

```
<img src="https://raw.githubusercontent.com/Markus-DS-29/Data-Cleaning-and-Storytelling/main/Weekly%20Revenue%20and%20Discount%20Percentage.png" alt="Weekly Revenue and Discount Percentage (Mean)" style="width: 892px; height: auto;">


## Analyse Discounts

```
# Plot discounts percentage of major categories over time
grouped_categories_disc_perc = analysis_categories_df.groupby(['date', 'category'])['discount_perc'].mean().unstack()

# Reset index to have 'date' as a column
grouped_categories_disc_perc = grouped_categories_disc_perc.reset_index()

# Resample the data to weekly frequency
resampled_disc_perc = grouped_categories_disc_perc.resample('W', on='date')[category_list].mean()

# Apply a rolling mean to smooth the data (e.g., using a 4-week window)
smoothed_disc_perc = resampled_disc_perc.rolling(window=4, min_periods=1).mean()

# Filter the data to start from May 2017
filtered_disc_perc = smoothed_disc_perc[smoothed_disc_perc.index >= '2017-05-01']

# Plot the filtered and smoothed data
fig, ax = plt.subplots(figsize=(10, 4))

for category in category_list:
    ax.plot(filtered_disc_perc.index, filtered_disc_perc[category], color='lightgray')  # Plot without labels

    # Fit a linear regression model to the category data
    x_values = np.arange(len(filtered_disc_perc[category].dropna()))
    y_values = filtered_disc_perc[category].dropna()
    slope, intercept = np.polyfit(x_values, y_values, 1)

    # Plot the trendline for the category
    trendline = slope * x_values + intercept
    ax.plot(filtered_disc_perc[category].dropna().index, trendline, linestyle='--', label=f'{category} Trendline')

# Set plot title and labels
plt.title('Discount Percentage of Major Categories Over Time')
plt.xlabel('Date')
plt.ylabel('Discount Percentage')

# Show the legend for trendlines in the top left position
plt.legend(loc='upper left')

# Add a grid
ax.grid(True)

plt.show()
```

<img src="https://raw.githubusercontent.com/Markus-DS-29/Data-Cleaning-and-Storytelling/main/Discount%20Percentage%20of%20Major%20Categories%20Over%20Time.png" alt="Discount Percentage of Major Categories Over Time" style="width: 892px; height: auto;">
