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

