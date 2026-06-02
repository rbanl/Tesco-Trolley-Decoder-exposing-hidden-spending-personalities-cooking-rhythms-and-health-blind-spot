# London Trolley Decoder
Decoding years of anonymised London customer receipts with NLP to expose hidden spending personalities, cooking rhythms, and health blind spots

---

## Overview

We all shop for groceries. But what do years of a single anonymised London customer’s receipts really reveal about their life?  

London Grocery Trolley Decoder took multiple years of real‑world grocery transaction data — messy product names, timestamps, prices — and transformed them into a rich, data‑driven portrait of that individual’s shopping behaviour. Every product was cleaned, classified, and analysed using a custom NLP pipeline, surfacing the hidden stories behind the trolley.

The result is a set of story‑driven charts that turn years of mundane receipts into a revealing mirror of everyday habits — and into a strategic playbook for how a retailer might better serve a loyal customer.

---

## Key Insights Uncovered (Customer)

- Temporal spending patterns reveal a routine‑driven structure, likely shaped around work and home location.  
- Store concentration implies strong geographic attachment and predictable routes — especially to a major London superstore.  
- Category analysis shows the customer enjoys savoury foods moderately high in fat, with a clear preference for red meat and cheese.  
- Despite those strong savoury preferences, overall purchasing behaviour still suggests moderate dietary balance — the basket is not extreme.

---

## Potential Commercial Value for a Grocery Retailer

- Highly suitable for personalised promotions due to consistent and predictable shopping routines.  
- Strong superstore affinity creates an opportunity for store‑specific offers and localised campaigns.  
- Weekend purchasing behaviour enables time‑targeted recommendations and promotions.  
- A food‑centric basket composition creates natural cross‑sell opportunities within complementary grocery categories.  
- Established preferences support tailored product recommendations and basket‑expansion strategies.  

---

All of this was built from raw receipt data without any pre‑existing labels or machine‑learning training — the classification is entirely rule‑based NLP, making the methodology transparent, interpretable, and surprisingly accurate.

[Download the PowerPoint presentation slides]https://github.com/rbanl/London-Trolley-Decoder-exposing-hidden-spending-personalities-cooking-rhythms-and-health-blind-spot/blob/main/London%20Customer%20Analysis.pptx

## Python Code

```python
# Paste your full analysis script here

[Download the PowerPoint presentation slides]https://github.com/rbanl/Tesco-Trolley-Decoder-exposing-hidden-spending-personalities-cooking-rhythms-and-health-blind-spot/raw/refs/heads/main/Tesco%20Customer%20Analysis.pptx

##Please see python code below:


import re
import pandas as pd
from wordsegment import load, segment
%cd "C:/rober/YourName/OneDrive/Documents/Work + job applications/Python Projects/AECOM Analysis/

load()

def clean_column_name(col: str) -> str:
    # 1. Replace dots with underscores
    col = col.replace('.', '_')

    # 2. Split CamelCase/PascalCase
    col = re.sub(r'([a-z0-9])([A-Z])', r'\1_\2', col)
    col = re.sub(r'([A-Z]+)([A-Z][a-z])', r'\1_\2', col)

    # 3. Lowercase concatenated words like "storeaddress"
    if '_' not in col and col.islower():
        words = segment(col)
        col = '_'.join(words)

    # 4. Final standardisation
    col = col.lower()
    col = re.sub(r'_+', '_', col)   # collapse multiple underscores
    col = col.strip('_')

    # 5. Remove the word "value" wherever it appears as a whole token
    parts = col.split('_')
    parts = [p for p in parts if p != 'value']   # drop exactly "value"
    col = '_'.join(parts)
    col = re.sub(r'_+', '_', col).strip('_')     # clean up again

    return col

# Apply to your DataFrame
tesco_data.columns = [clean_column_name(c) for c in tesco_data.columns]

print(tesco_data.columns)

import pandas as pd
import matplotlib.pyplot as plt

# Pivot table: sum of (product_price * product_quantity) per store_id
pivot_price = (
    tesco_data.groupby('store_id', as_index=False)
    .agg(total_price=('product_price', lambda x: (x * tesco_data.loc[x.index, 'product_quantity']).sum()))
)

# Sort and take top 10
top10_price = pivot_price.nlargest(10, 'total_price')

# Sort ascending for horizontal bar (largest at the top)
top10_price = top10_price.sort_values('total_price', ascending=True)

# --- Colour palette ---
colors = ['#1A237E', '#283593', '#1565C0', '#42A5F5', '#00897B',
          '#00ACC1', '#FF6F61', '#C44E52', '#5E35B1', '#FFA726']

# Plot – horizontal bars (store_id as string to avoid numeric coordinates)
fig, ax = plt.subplots(figsize=(12, 7))
bars = ax.barh(top10_price['store_id'].astype(str),   # <--- FIX: convert to string
               top10_price['total_price'],
               color=colors[:len(top10_price)],
               edgecolor='white', linewidth=1.2)

# Add spend labels
for bar, val in zip(bars, top10_price['total_price']):
    ax.text(bar.get_width() + max(top10_price['total_price'])*0.01,
            bar.get_y() + bar.get_height()/2,
            f'£{val:,.0f}', va='center', fontsize=14, fontweight='bold')

# Styling
ax.set_title('Money Spent at Different Tesco Stores', fontsize=16, weight='bold', pad=15)
ax.set_xlabel('Total Spend (£)', fontsize=14)
ax.set_ylabel('Tesco Store', fontsize=16)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.xaxis.grid(True, linestyle='--', alpha=0.4)
ax.set_axisbelow(True)
ax.set_xlim(0, max(top10_price['total_price']) * 1.2)
ax.tick_params(axis='y', labelsize=16)

plt.tight_layout()
plt.show()
import pandas as pd
import matplotlib.pyplot as plt

# Pivot table: unique time_stamp count per store_id
pivot_visits = (
    tesco_data.groupby('store_id', as_index=False)
    .agg(unique_visits=('time_stamp', 'nunique'))
)

# Sort and take top 10
top10_visits = pivot_visits.nlargest(10, 'unique_visits')

# Sort ascending for horizontal bar (largest at top)
top10_visits = top10_visits.sort_values('unique_visits', ascending=True)

# --- Colour palette ---
colors = ['#1A237E', '#283593', '#1565C0', '#42A5F5', '#00897B',
          '#00ACC1', '#FF6F61', '#C44E52', '#5E35B1', '#FFA726']

# Plot – horizontal bars (store_id as string so bars appear correctly)
fig, ax = plt.subplots(figsize=(12, 7))
bars = ax.barh(top10_visits['store_id'].astype(str),   # numeric → string
               top10_visits['unique_visits'],
               color=colors[:len(top10_visits)], edgecolor='white', linewidth=1.2)

# Add value labels at the end of each bar
for bar, val in zip(bars, top10_visits['unique_visits']):
    ax.text(bar.get_width() + max(top10_visits['unique_visits'])*0.01,
            bar.get_y() + bar.get_height()/2,
            f'{val}', va='center', fontsize=14, fontweight='bold')

# Styling
ax.set_title('Number of Basket Purchases at Different Tesco Stores', fontsize=16, weight='bold', pad=15)
ax.set_xlabel('Basket Purchases', fontsize=14)
ax.set_ylabel('Tesco Store', fontsize=14)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.xaxis.grid(True, linestyle='--', alpha=0.4)
ax.set_axisbelow(True)
ax.set_xlim(0, max(top10_visits['unique_visits']) * 1.2)   # room for labels
ax.tick_params(axis='y', labelsize=16)

plt.tight_layout()
plt.show()
tesco_data['time_stamp'] = pd.to_datetime(tesco_data['time_stamp'])
tesco_data['shopping_period'] = tesco_data['time_stamp'].dt.to_period('M')  # monthly

# Aggregate total sales per period
period_price = (
    tesco_data.groupby('shopping_period', as_index=False)
    .agg(
        total_price=('product_price', lambda x: (x * tesco_data.loc[x.index, 'product_quantity']).sum())
    )
)

# Get top 10
top10_price_periods = period_price.nlargest(10, 'total_price')

# Plot
fig, ax = plt.subplots(figsize=(10, 6))
bars = ax.bar(top10_price_periods['shopping_period'].astype(str),
              top10_price_periods['total_price'],
              color='steelblue', edgecolor='grey', alpha=0.85)

ax.set_title('Top 10 Shopping Periods by Total Spend', fontsize=14, weight='bold')
ax.set_xlabel('Date')
ax.set_ylabel('Total Spend (£)')
ax.tick_params(axis='x', rotation=45)
plt.tight_layout()
plt.show()
# Count the number of rows (transactions) per period
period_visits = (
    tesco_data.groupby('shopping_date', as_index=False)
    .agg(transaction_count=('time_stamp', 'count'))  # each row is a timestamp
)

# Top 10
top10_visits_periods = period_visits.nlargest(10, 'transaction_count')

# Plot (different colour)
fig, ax = plt.subplots(figsize=(10, 6))
bars = ax.bar(top10_visits_periods['shopping_date'].astype(str),
              top10_visits_periods['transaction_count'],
              color='coral', edgecolor='grey', alpha=0.85)
ax.set_title('Top 10 Shopping Days by Number of Transactions', fontsize=14, weight='bold')
ax.set_xlabel('Date')
ax.set_ylabel('Number of Timestamps (Transactions)')
ax.tick_params(axis='x', rotation=45)
plt.tight_layout()
plt.show()



import pandas as pd
import matplotlib.pyplot as plt

# Aggregate total spend (price × quantity) per day of week
dow_spend = (
    tesco_data.groupby('day_of_week', as_index=False)
    .agg(
        total_price=('product_price', lambda x: (x * tesco_data.loc[x.index, 'product_quantity']).sum())
    )
)

# Sort descending (highest spend first)
dow_spend = dow_spend.sort_values('total_price', ascending=False)

# --- Colour palette (one rich colour per bar) ---
day_colors = ['#1A237E', '#1565C0', '#00897B', '#00ACC1', '#FF6F61',
              '#C44E52', '#5E35B1'][:len(dow_spend)]

# Plot – vertical bars
fig, ax = plt.subplots(figsize=(12, 7))
bars = ax.bar(dow_spend['day_of_week'], dow_spend['total_price'],
              color=day_colors, edgecolor='white', linewidth=1.2)

# Spend labels on top of each bar
for bar, val in zip(bars, dow_spend['total_price']):
    ax.text(bar.get_x() + bar.get_width()/2,
            bar.get_height() + max(dow_spend['total_price'])*0.02,
            f'{val:,.0f}', ha='center', fontsize=14, fontweight='bold')

# Styling (matching your reference)
ax.set_title('Total Spend by Day of Week', fontsize=16, weight='bold', pad=15)
ax.set_xlabel('Day of Week', fontsize=14)
ax.set_ylabel('Total Spend (£)', fontsize=14)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.yaxis.grid(True, linestyle='--', alpha=0.4)
ax.set_axisbelow(True)
ax.tick_params(axis='x', rotation=0, labelsize=16)   # days are short, no rotation needed
ax.tick_params(axis='y', labelsize=14)
ax.set_ylim(0, max(dow_spend['total_price']) * 1.15)   # headroom for labels

plt.tight_layout()
plt.show()

import re
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from collections import defaultdict

# ----------------------------------------------------------------------
# 0. COPY DATA
# ----------------------------------------------------------------------
df = tesco_data.copy()

# ----------------------------------------------------------------------
# 1. CLEAN PRODUCT NAMES
# ----------------------------------------------------------------------
def clean_product_name(text):
    text = str(text).lower()
    # remove weights / measures (e.g. 100g, 2.5kg, 500ml)
    text = re.sub(r'\b\d+(\.\d+)?\s?(g|kg|ml|l|lt|cl|pack|pk)\b', ' ', text)
    # remove punctuation except spaces
    text = re.sub(r'[^a-zA-Z\s]', ' ', text)
    # collapse multiple spaces
    text = re.sub(r'\s+', ' ', text).strip()
    return text

df['product_clean'] = df['product_name'].apply(clean_product_name)

# ----------------------------------------------------------------------
# 2. BRAND HINTS — if a brand is found, the corresponding category
#    receives a large bonus score (20), almost guaranteeing correct placement.
# ----------------------------------------------------------------------
BRAND_HINTS = {
    "fairy": "Household Cleaning",
    "fever tree": "Soft Drinks & Mixers",
    "funko": "Toys & Gifts",
    "good boy": "Pet Care",
    "flo": "Feminine Care",
    "febreze": "Household Cleaning",
    "persil": "Household Cleaning",
    "ariel": "Household Cleaning",
    "bold": "Household Cleaning",
    "lenor": "Household Cleaning",
    "coca cola": "Soft Drinks & Mixers",
    "pepsi": "Soft Drinks & Mixers",
    "red bull": "Soft Drinks & Mixers",
    "monster": "Soft Drinks & Mixers",
    "innocent": "Drinks",
    "tropicana": "Drinks",
    "alpro": "Dairy",   # plant-based often near dairy
    "oatly": "Dairy",
    "cathedral": "Dairy",
}

BRAND_SCORE = 20   # high enough to dominate any keyword matches

# ----------------------------------------------------------------------
# 3. CATEGORY KEYWORDS WITH WEIGHTS
#    Phrase bonus is added automatically later: +2 per extra word.
#    Eggs have been merged into Meat & Poultry.
# ----------------------------------------------------------------------
CATEGORY_KEYWORDS = {

    "Meat & Poultry": {
        "chicken": 5, "beef": 5, "pork": 5, "lamb": 5, "turkey": 5,
        "duck": 5, "bacon": 5, "sausage": 5, "ham": 5, "mince": 5,
        "steak": 5, "fillet": 4, "fillets": 4, "breast": 4,
        "drumstick": 4, "goujon": 4, "goujons": 4, "roast": 4,
        "joint": 4,
        # Eggs merged here
        "egg": 5, "eggs": 5, "omelette": 4, "free range": 3
    },

    "Vegetables & Salad": {
        "broccoli": 5, "spinach": 5, "carrot": 4, "carrots": 4,
        "onion": 4, "onions": 4, "mushroom": 4, "mushrooms": 4,
        "lettuce": 4, "salad": 5, "vegetable": 4, "vegetables": 4,
        "rocket": 4, "watercress": 4, "kale": 4, "pepper": 3,
        "peppers": 3, "courgette": 4, "tomato": 4, "tomatoes": 4,
        "potato": 3, "potatoes": 3, "greens": 3, "beetroot": 4,
        "avocado": 4, "pak choi": 5, "spring onion": 4, "asparagus": 4
    },

    "Fruit": {
        "apple": 5, "banana": 5, "orange": 5, "clementine": 5,
        "berries": 5, "berry": 5, "grape": 5, "melon": 5, "kiwi": 5,
        "mango": 5, "pineapple": 5, "fruit": 4, "peeler": 3,
        "lemon": 4, "lime": 4, "pear": 4
    },

    "Fish & Seafood": {
        "salmon": 5, "tuna": 5, "prawn": 5, "prawns": 5, "cod": 5,
        "haddock": 5, "mackerel": 5, "fish": 5, "seafood": 5,
        "sea bass": 5, "trout": 5
    },

    "Dairy": {
        "milk": 5, "cream": 4, "butter": 5, "yoghurt": 5, "yogurt": 5,
        "cheese": 5, "cheddar": 5, "mozzarella": 5, "feta": 5, "brie": 5,
        "camembert": 5, "philadelphia": 5, "leerdammer": 5, "parmesan": 5
    },

    "Pasta & Grains": {
        "pasta": 5, "spaghetti": 5, "rigatoni": 5, "penne": 5,
        "fusilli": 5, "macaroni": 5, "rice": 3, "quinoa": 5,
        "couscous": 5, "bulgur": 5, "grain": 4, "grains": 4,
        "noodle": 5, "noodles": 5
    },

    "Cereal & Breakfast": {
        "cereal": 5, "granola": 5, "muesli": 5, "porridge": 5,
        "oat": 4, "oats": 4, "weetabix": 5, "shreddies": 5, "cheerios": 5
    },

    "Bakery & Bread": {
        "bread": 5, "roll": 4, "rolls": 4, "baguette": 5, "croissant": 5,
        "muffin": 5, "cake": 5, "pastry": 5, "wrap": 5, "wraps": 5,
        "tortilla": 5, "pitta": 5, "naan": 5, "crumpet": 5
    },

    "Snacks & Confectionery": {
        "crisp": 5, "crisps": 5, "popcorn": 5, "chocolate": 5,
        "biscuit": 5, "biscuits": 5, "cookie": 5, "cookies": 5,
        "brownie": 5, "sweet": 4, "sweets": 4, "candy": 5,
        "snack": 5, "bar": 3, "seaweed": 4, "almond": 3,
        "almonds": 3, "cashew": 3, "nuts": 3, "guacamole": 5,
        "salsa": 5, "tortilla chip": 5
    },

    "Frozen Foods": {
        "frozen": 5, "pizza": 5, "chips": 5, "waffles": 4,
        "fish fingers": 5, "ice cream": 5, "ice lolly": 5,
        "sorbet": 5, "gelato": 5
    },

    "Sauces & Condiments": {
        "sauce": 5, "passata": 5, "vinegar": 5, "oil": 5,
        "dressing": 5, "ketchup": 5, "mayonnaise": 5, "mayo": 5,
        "dip": 5, "pesto": 5, "stock": 4, "gravy": 4,
        "harissa": 5, "mustard": 5, "soy sauce": 5, "worcestershire": 5,
        "barbecue sauce": 5
    },

    "Drinks": {
        "water": 5, "juice": 5, "smoothie": 5, "coffee": 5,
        "tea": 5, "cola": 5, "drink": 4, "sparkling": 4,
        "cordial": 5, "squash": 5, "tonic": 5, "lemonade": 5,
        "ginger beer": 5, "fizzy": 4
    },

    "Soft Drinks & Mixers": {
        "tonic water": 5, "fever tree": 5, "ginger ale": 5,
        "mixer": 5, "club soda": 5
    },

    "Alcohol": {
        "wine": 5, "beer": 5, "cider": 5, "lager": 5, "ale": 5,
        "vodka": 5, "whisky": 5, "whiskey": 5, "rum": 5, "gin": 5,
        "brandy": 5, "champagne": 5, "prosecco": 5, "rosé": 5, "rose wine": 5
    },

    "Baking Ingredients": {
        "flour": 5, "sugar": 5, "yeast": 5, "vanilla": 4, "cocoa": 5,
        "baking powder": 5, "ground almonds": 5, "sweetener": 4, "stevia": 5,
        "marzipan": 5, "icing": 5
    },

    "Desserts & Sweet Treats": {
        "dessert": 5, "panna cotta": 5, "mousse": 5, "cheesecake": 5,
        "profiterole": 5, "tiramisu": 5, "crème brûlée": 5,
        "pudding": 5, "sponge": 4, "custard": 4
    },

    "Household Cleaning": {
        "detergent": 5, "bleach": 5, "cleaning": 5, "spray": 4,
        "wipe": 4, "ironing water": 5, "fairy": 5, "febreze": 5,
        "persil": 5, "ariel": 5, "bold": 5, "lenor": 5,
        "dishwasher": 5, "washing up liquid": 5, "floor cleaner": 5
    },

    "Homeware & Lifestyle": {
        "candle": 5, "mug": 4, "homeware": 5, "apothecary": 4,
        "vase": 4, "frame": 4, "ornament": 4, "decor": 4,
        "furniture": 5, "cushion": 5, "rug": 5, "lamp": 4
    },

    "Toiletries & Personal Care": {
        "toothpaste": 5, "shampoo": 5, "conditioner": 5,
        "deodorant": 5, "shower gel": 5, "soap": 5,
        "handwash": 5, "razor": 5, "blade": 5, "sanitary": 5,
        "nappy": 5, "tampon": 5, "pads": 5, "winged": 4,
        "ultra thin": 4, "liners": 5, "cotton wool": 5,
        "facial tissue": 5
    },

    "Feminine Care": {
        "feminine": 5, "period": 5, "menstrual": 5,
        "panty liner": 5, "sanitary towel": 5, "tampax": 5
    },

    "Pet Care": {
        "dog": 5, "cat": 5, "pet": 4, "poo bags": 5,
        "training pads": 5, "litter": 5, "food for dogs": 5,
        "biscuit for dogs": 5, "cat food": 5
    },

    "Flowers & Plants": {
        "bouquet": 5, "rose": 4, "lily": 4, "flower": 5,
        "plant": 5, "orchid": 5, "tulip": 5, "daffodil": 5,
        "succulent": 5, "foliage": 4
    },

      "Male Care": {
        "men": 5, "male": 5, "shaving": 5, "shave": 5,
        "aftershave": 5, "beard": 5, "just for men": 5,
        "sure men": 5
    },

    "Toys & Gifts": {
        "toy": 5, "plush": 5, "funko": 5, "figure": 5,
        "gift": 4, "collectible": 5, "card game": 5,
        "puzzle": 5, "board game": 5, "soft toy": 5
    },

    "Ready-to-Drink Coffee": {
        "latte": 5, "cappuccino": 5, "flat white": 5,
        "iced coffee": 5, "espresso shot": 5, "coffee drink": 5,
        "mocha": 5, "americano": 5
    },
}

# ----------------------------------------------------------------------
# 4. CATEGORY PRIORITIES — used to break ties
#    (Eggs removed, now part of Meat & Poultry)
# ----------------------------------------------------------------------
CATEGORY_PRIORITY = [
    "Fish & Seafood",
    "Meat & Poultry",            # Eggs will now be classified here
    "Vegetables & Salad",
    "Fruit",
    "Dairy",
    "Frozen Foods",
    "Sauces & Condiments",
    "Pasta & Grains",
    "Cereal & Breakfast",
    "Bakery & Bread",
    "Desserts & Sweet Treats",
    "Snacks & Confectionery",
    "Ready-to-Drink Coffee",
    "Soft Drinks & Mixers",
    "Drinks",
    "Alcohol",
    "Baking Ingredients",
    "Household Cleaning",
    "Homeware & Lifestyle",
    "Toiletries & Personal Care",
    "Feminine Care",
    "Pet Care",
    "Male Care",
    "Flowers & Plants",
    "Toys & Gifts",
]

# ----------------------------------------------------------------------
# 5. SCORING FUNCTION WITH BRAND HINTS & PHRASE BONUS
# ----------------------------------------------------------------------
PHRASE_BONUS_PER_EXTRA_WORD = 2   # added for each word beyond the first

def classify_product(product_name):
    text = product_name.lower()

    scores = defaultdict(int)

    # --- brand hints: huge bonus ---
    for brand, cat in BRAND_HINTS.items():
        if brand in text:
            scores[cat] += BRAND_SCORE
            # break? some products might match multiple brands, we add all

    # --- regular keyword matching with phrase bonus ---
    for category, keywords in CATEGORY_KEYWORDS.items():
        for keyword, weight in keywords.items():
            if keyword in text:
                # phrase bonus: number of words in keyword - 1
                extra_words = keyword.count(' ')  # spaces, so words = spaces+1
                bonus = extra_words * PHRASE_BONUS_PER_EXTRA_WORD
                scores[category] += weight + bonus

    # no matches at all → Other
    if len(scores) == 0:
        return "Other"

    # highest score
    max_score = max(scores.values())
    top_categories = [cat for cat, score in scores.items() if score == max_score]

    # tie-break using priority order
    for priority_cat in CATEGORY_PRIORITY:
        if priority_cat in top_categories:
            return priority_cat

    return top_categories[0]

# ----------------------------------------------------------------------
# 6. APPLY CLASSIFICATION
# ----------------------------------------------------------------------
df['category'] = df['product_clean'].apply(classify_product)

# ----------------------------------------------------------------------
# 7. PRINT PRODUCT MAPPINGS
# ----------------------------------------------------------------------
print("\n=== PRODUCT → CATEGORY MAPPING ===\n")

mapping_df = (
    df[['product_clean', 'category']]
    .drop_duplicates()
    .sort_values(['category', 'product_clean'])
)

for _, row in mapping_df.iterrows():
    print(f"{row['product_clean']:<65} → {row['category']}")

# ----------------------------------------------------------------------
# 8. CATEGORY SPEND ANALYSIS
# ----------------------------------------------------------------------
category_spend = (
    df.groupby('category', as_index=False)
    .agg(
        total_spend=('product_price', lambda x: (x * df.loc[x.index, 'product_quantity']).sum()),
        unique_products=('product_clean', 'nunique')
    )
    .sort_values('total_spend', ascending=False)
)
# ----------------------------------------------------------------------
# 9. VISUALISATION  –  BIG FONTS, UNIQUE COLOURS, NO LEGEND
# ----------------------------------------------------------------------
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter

# Sort ascending so largest bar sits at the bottom
category_spend_sorted = category_spend.sort_values('total_spend', ascending=True)
labels  = category_spend_sorted['category'].tolist()
values  = category_spend_sorted['total_spend'].tolist()
total   = sum(values)
n       = len(labels)

# Distinct colour palette – each bar gets a unique colour
UNIQUE_COLORS = [
    '#e6194B', '#3cb44b', '#ffe119', '#4363d8', '#f58231',
    '#911eb4', '#42d4f4', '#f032e6', '#bfef45', '#fabed4',
    '#469990', '#dcbeff', '#9A6324', '#fffac8', '#800000',
    '#aaffc3', '#808000', '#ffd8b1', '#000075', '#a9a9a9',
    '#FF6F61', '#6B5B95', '#88B04B', '#F7CAC9', '#92A8D1',
    '#955251', '#B565A7', '#009B77', '#DD4124', '#D65076'
]
# Cycle colours if there are more categories than colours
colors = [UNIQUE_COLORS[i % len(UNIQUE_COLORS)] for i in range(n)]

fig, ax = plt.subplots(figsize=(13, 8))
fig.patch.set_facecolor('#F5F5F5')
ax.set_facecolor('#F5F5F5')

bars = ax.barh(labels, values, color=colors, edgecolor='none', height=0.65)

# Value labels at the end of each bar
for bar, val in zip(bars, values):
    ax.text(
        bar.get_width() + max(values) * 0.01,
        bar.get_y() + bar.get_height() / 2,
        f'£{val:,.0f}',
        va='center', ha='left', fontsize=14,
        color='#222222'
    )

ax.set_xlim(0, max(values) * 1.22)
ax.xaxis.grid(True, color='white', linewidth=1.4, zorder=0)
ax.set_axisbelow(True)
ax.spines[['top', 'right', 'left', 'bottom']].set_visible(False)
ax.tick_params(length=0)
ax.tick_params(axis='y', labelsize=16, pad=8)
ax.tick_params(axis='x', labelsize=14)

def gbp_fmt(x, _):
    if x == 0:    return '0'
    if x < 1000:  return f'{x:.0f}'
    return f'{x/1000:.1f}k'.replace('.0k', 'k')

ax.xaxis.set_major_formatter(FuncFormatter(gbp_fmt))
ax.set_xlim(0, max(values) * 1.05)

ax.set_title('Total Spend across Item Categories', fontsize=22, weight='bold', pad=20)
ax.set_xlabel('Total Spend (£)', fontsize=16, labelpad=10)
ax.set_ylabel('')

plt.tight_layout()
fig.savefig('categoryplot2.png', dpi=300, bbox_inches='tight')
plt.show()

total_spend = category_spend['total_spend'].sum()
print(f"Total spend across all categories: £{total_spend:,.2f}")

# ======================================================================
# DAIRY + MEAT SUBCATEGORY ANALYSIS (IMPROVED VERSION)
# ======================================================================

import pandas as pd
import matplotlib.pyplot as plt
from collections import defaultdict

# ----------------------------------------------------------------------
# 1. FILTER TO MAIN CATEGORIES
# ----------------------------------------------------------------------
df_sub = df[df['category'].isin([
    'Dairy',
    'Meat & Poultry'
])].copy()

# ----------------------------------------------------------------------
# 2. DAIRY SUBCATEGORIES
# ----------------------------------------------------------------------
DAIRY_SUBCATEGORIES = {

    "Milk": {
        "milk": 6,
        "whole milk": 7,
        "semi skimmed": 7,
        "skimmed": 7,
        "oat milk": 8,
        "almond milk": 8
    },

    "Cheese": {
        "cheese": 6,
        "cheddar": 8,
        "mozzarella": 8,
        "brie": 8,
        "camembert": 8,
        "feta": 8,
        "parmesan": 8,
        "philadelphia": 7,
        "leerdammer": 7,
        "halloumi": 8,
        "gorgonzola": 8
    },

    "Yoghurt": {
        "yoghurt": 7,
        "yogurt": 7,
        "greek yoghurt": 9,
        "greek yogurt": 9,
        "protein yoghurt": 9,
        "protein yogurt": 9,
        "skyr": 9
    },

    "Cream & Butter": {
        "cream": 6,
        "double cream": 9,
        "single cream": 9,
        "extra thick cream": 10,
        "butter": 8,
        "spread": 5
    },

    # NEW + IMPROVED
    "Sweet Dairy & Desserts": {

        # dessert products
        "dessert": 8,
        "pudding": 8,
        "custard": 8,
        "mousse": 8,
        "cheesecake": 10,
        "ice cream": 10,
        "gelato": 10,
        "sorbet": 9,
        "panna cotta": 10,

        # sweet dairy drinks
        "milkshake": 8,
        "frappe": 8,
        "latte": 6,
        "caramel": 5,

        # sweet yoghurt indicators
        "flavoured yoghurt": 8,
        "flavored yoghurt": 8,
        "vanilla": 5,
        "chocolate": 5,
        "strawberry": 5,
        "sweet": 4
    }
}

# ----------------------------------------------------------------------
# 3. MEAT SUBCATEGORIES
# ----------------------------------------------------------------------
MEAT_SUBCATEGORIES = {

    # NEW
    "Eggs": {
        "egg": 10,
        "eggs": 10,
        "free range": 5,
        "omelette": 7
    },

    "Chicken": {
        "chicken": 8,
        "breast": 6,
        "drumstick": 6,
        "thigh": 6,
        "goujon": 7,
        "goujons": 7
    },

    "Beef": {
        "beef": 8,
        "steak": 8,
        "mince": 7
    },

    "Pork & Bacon": {
        "pork": 8,
        "bacon": 8,
        "ham": 7,
        "sausage": 7
    },

    "Premium Cuts": {
        "sirloin": 10,
        "ribeye": 10,
        "fillet": 8
    },

    "Processed / Convenience Meat": {
        "breaded": 8,
        "frozen": 7,
        "ready meal": 8,
        "processed": 7
    }
}

# ----------------------------------------------------------------------
# 4. PRIORITY ORDERS
# ----------------------------------------------------------------------
DAIRY_PRIORITY = [

    "Cheese",

    # moved above yoghurt
    "Sweet Dairy & Desserts",

    "Yoghurt",
    "Cream & Butter",
    "Milk"
]

MEAT_PRIORITY = [

    # NEW
    "Eggs",

    "Chicken",
    "Beef",
    "Pork & Bacon",
    "Premium Cuts",
    "Processed / Convenience Meat"
]

# ----------------------------------------------------------------------
# 5. GENERIC CLASSIFIER
# ----------------------------------------------------------------------
def classify_subcategory(product_name, taxonomy, priority_order):

    text = str(product_name).lower()

    scores = defaultdict(int)

    # score categories
    for category, keywords in taxonomy.items():

        for keyword, weight in keywords.items():

            if keyword in text:
                scores[category] += weight

    # no matches
    if len(scores) == 0:
        return "Other"

    # best score
    max_score = max(scores.values())

    top_categories = [
        cat for cat, score in scores.items()
        if score == max_score
    ]

    # priority tie-break
    for cat in priority_order:
        if cat in top_categories:
            return cat

    return top_categories[0]

# ----------------------------------------------------------------------
# 6. APPLY SUBCATEGORY CLASSIFICATION
# ----------------------------------------------------------------------
def assign_subcategory(row):

    category = row['category']
    product = row['product_clean']

    if category == 'Dairy':

        return classify_subcategory(
            product,
            DAIRY_SUBCATEGORIES,
            DAIRY_PRIORITY
        )

    elif category == 'Meat & Poultry':

        return classify_subcategory(
            product,
            MEAT_SUBCATEGORIES,
            MEAT_PRIORITY
        )

    return None

df_sub['subcategory'] = df_sub.apply(assign_subcategory, axis=1)

# ----------------------------------------------------------------------
# 7. PRINT PRODUCT MAPPINGS
# ----------------------------------------------------------------------
print("\n==============================")
print("PRODUCT → SUBCATEGORY")
print("==============================\n")

mapping_df = (
    df_sub[['product_clean', 'category', 'subcategory']]
    .drop_duplicates()
    .sort_values(['category', 'subcategory', 'product_clean'])
)

for _, row in mapping_df.iterrows():

    print(
        f"{row['product_clean']:<65} "
        f"→ {row['category']:<20} "
        f"→ {row['subcategory']}"
    )

# ----------------------------------------------------------------------
# 8. SUBCATEGORY SPEND ANALYSIS (CORRECTED)
# ----------------------------------------------------------------------
subcategory_spend = (
    df_sub.groupby(
        ['category', 'subcategory'],
        as_index=False
    )
    .agg(
        total_spend=('product_price', lambda x: (x * df_sub.loc[x.index, 'product_quantity']).sum()),
        unique_products=('product_clean', 'nunique')
    )
    .sort_values(
        ['category', 'total_spend'],
        ascending=[True, False]
    )
)

print("\n==============================")
print("SUBCATEGORY SPEND")
print("==============================\n")

print(subcategory_spend)

# ======================================================================
# 9. BEAUTIFUL PIE CHARTS (percentage‑based, saved as PNG)
# ======================================================================
import os

plt.style.use('seaborn-v0_8-white')

# Create folder for the images
os.makedirs('plots', exist_ok=True)

# Distinct, vibrant colour sets
dairy_colors = ['#1A237E', '#1565C0', '#00897B', '#00ACC1', '#FF6F61']
meat_colors  = ['#C44E52', '#FFA726', '#5E35B1', '#42A5F5', '#FF6F61', '#00ACC1']

category_colors = {
    'Dairy': dairy_colors,
    'Meat & Poultry': meat_colors
}

for main_category in ['Dairy', 'Meat & Poultry']:

    plot_df = subcategory_spend[
        subcategory_spend['category'] == main_category
    ].copy()

    if len(plot_df) == 0:
        continue

    # Calculate percentages for each subcategory
    total_spend = plot_df['total_spend'].sum()
    plot_df['pct_spend'] = (plot_df['total_spend'] / total_spend) * 100

    # Colours for this category – loop if more subcats than colours
    cat_colors = category_colors[main_category]
    wedge_colors = cat_colors[:len(plot_df)] if len(plot_df) <= len(cat_colors) else (
        cat_colors * (len(plot_df)//len(cat_colors) + 1))[:len(plot_df)]

    # Explode the largest slice slightly for emphasis
    explode_vals = [0.0] * len(plot_df)
    max_idx = plot_df['pct_spend'].idxmax()
    # convert to positional index
    max_pos = plot_df.index.get_loc(max_idx)
    explode_vals[max_pos] = 0.07   # slight pull‑out

    # ----- CREATE THE PIE CHART -----
    fig, ax = plt.subplots(figsize=(12, 10))

    wedges, texts, autotexts = ax.pie(
        plot_df['pct_spend'],
        labels=plot_df['subcategory'],
        autopct='%1.1f%%',
        startangle=90,
        explode=explode_vals,
        colors=wedge_colors,
        pctdistance=0.85,
        wedgeprops={'edgecolor': 'white', 'linewidth': 1.5},
        textprops={'fontsize': 13},
    )

    # Style the percentage labels inside the slices
    for autotext in autotexts:
        autotext.set_fontsize(12)
        autotext.set_fontweight('bold')
        autotext.set_color('white')

    # Title with category
    ax.set_title(
        f'{main_category} – Where the money goes',
        fontsize=18, weight='bold', pad=25
    )

    # Equal aspect ratio ensures the pie is drawn as a circle
    ax.axis('equal')

    plt.tight_layout()

    # ----- SAVE TO PNG -----
    safe_name = main_category.lower().replace(' & ', '_').replace(' ', '_')
    filepath = f'plots/{safe_name}_subcategory_spend_pie.png'
    plt.savefig(filepath, dpi=300, bbox_inches='tight')
    print(f'Saved: {filepath}')

    plt.show()
# ----------------------------------------------------------------------
# 10. BEHAVIOURAL INSIGHTS
# ----------------------------------------------------------------------
print("\n==============================")
print("BEHAVIOURAL INSIGHTS")
print("==============================\n")

# ----------------------------------------------------------------------
# DAIRY INSIGHTS
# ----------------------------------------------------------------------
dairy_df = subcategory_spend[
    subcategory_spend['category'] == 'Dairy'
]

if len(dairy_df) > 0:

    top_dairy = dairy_df.iloc[0]['subcategory']

    print(f"Top Dairy Preference: {top_dairy}")

    if top_dairy == 'Cheese':

        print("- Strong preference for savoury dairy products")
        print("- Suggests cooking-oriented purchasing behaviour")
        print("- Potential premium food preference")

    elif top_dairy == 'Yoghurt':

        print("- Suggests health-conscious purchasing")
        print("- Possible fitness / wellness orientation")

    elif top_dairy == 'Cream & Butter':

        print("- Suggests richer cooking ingredients")
        print("- Potential indulgence / premium cooking behaviour")

    elif top_dairy == 'Sweet Dairy & Desserts':

        print("- Suggests indulgent / treat-oriented purchasing")
        print("- Higher preference for sweet dairy products")
        print("- Potential snacking / dessert behaviour")

# ----------------------------------------------------------------------
# MEAT INSIGHTS
# ----------------------------------------------------------------------
meat_df = subcategory_spend[
    subcategory_spend['category'] == 'Meat & Poultry'
]

if len(meat_df) > 0:

    top_meat = meat_df.iloc[0]['subcategory']

    print(f"\nTop Meat Preference: {top_meat}")

    if top_meat == 'Eggs':

        print("- Strong breakfast/protein staple purchasing")
        print("- Suggests regular home meal preparation")

    elif top_meat == 'Chicken':

        print("- Lean protein preference")
        print("- Potential health-oriented purchasing")

    elif top_meat == 'Processed / Convenience Meat':

        print("- Convenience-oriented purchasing behaviour")

    elif top_meat == 'Premium Cuts':

        print("- Premium / higher-quality meat preference")

    elif top_meat == 'Pork & Bacon':

        print("- Traditional comfort-food purchasing patterns")

# ----------------------------------------------------------------------
# 9. PIE CHART VISUALISATIONS (UPGRADED DONUT CHARTS)
# ----------------------------------------------------------------------
import matplotlib.patheffects as path_effects
import matplotlib.pyplot as plt
import seaborn as sns

# Set a modern, clean minimal style
sns.set_style("white")
plt.rcParams['font.family'] = 'DejaVu Sans'

# A more vibrant, modern color palette
colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEEAD', '#D4A5A5', '#9B59B6']

for main_category in ['Dairy', 'Meat & Poultry']:

    plot_df = subcategory_spend[
        subcategory_spend['category'] == main_category
    ].copy()

    if len(plot_df) == 0:
        continue

    total_spend = plot_df['total_spend'].sum()
    plot_df['pct_spend'] = (plot_df['total_spend'] / total_spend) * 100

    # Automatically find the largest segment to "explode" (pop out)
    max_spend = plot_df['total_spend'].max()
    explode = [0.08 if spend == max_spend else 0 for spend in plot_df['total_spend']]

    fig, ax = plt.subplots(figsize=(10, 8))

    # Draw DONUT chart
    # The 'width' parameter is what turns the pie into a donut
    wedges, texts, autotexts = ax.pie(
        plot_df['total_spend'],
        labels=plot_df['subcategory'],
        autopct='%1.1f%%',
        startangle=140,
        pctdistance=0.75,         # Moves the percentage text into the middle of the donut ring
        explode=explode,          # Pops out the biggest slice
        colors=colors[:len(plot_df)],
        wedgeprops={'width': 0.45, 'edgecolor': 'white', 'linewidth': 2.5, 'alpha': 0.95},
        textprops={'fontsize': 16, 'fontweight': '500', 'color': '#2C3E50'}
    )

    # Style the percentage labels inside the rings
    for autotext in autotexts:
        autotext.set_color('white')
        autotext.set_fontweight('bold')
        autotext.set_fontsize(16)
        
        # Add a subtle text shadow to make white text pop on lighter colors
        autotext.set_path_effects([
            plt.matplotlib.patheffects.withStroke(linewidth=2, foreground='#333333', alpha=0.5)
        ])

    # Add dynamic text in the center of the donut
    center_text = f"{main_category}\nSpend"
    ax.text(0, 0, center_text, ha='center', va='center', fontsize=16, fontweight='bold', color='#333333')

    # Add a compelling title
    ax.set_title(
        f'Where the Money Goes: {main_category.upper()}',
        fontsize=22, 
        fontweight='900', 
        pad=20,
        color='#1A252F'
    )

    plt.tight_layout()
    plt.show()

import pandas as pd
import matplotlib.pyplot as plt

# ----------------------------------------------------------------------
# FILTER
# ----------------------------------------------------------------------
meat_df = df[df['category'] == 'Meat & Poultry'].copy()

# datetime
meat_df['time_stamp'] = pd.to_datetime(meat_df['time_stamp'])

# weekday
meat_df['day_of_week'] = meat_df['time_stamp'].dt.day_name()

weekday_order = [
    'Monday','Tuesday','Wednesday',
    'Thursday','Friday','Saturday','Sunday'
]

# ----------------------------------------------------------------------
# FREQUENCY
# ----------------------------------------------------------------------
meat_frequency = (
    meat_df.groupby('day_of_week')
    .size()
    .reindex(weekday_order, fill_value=0)
    .reset_index(name='purchase_count')
)

# percentage
total = meat_frequency['purchase_count'].sum()

meat_frequency['purchase_pct'] = (
    meat_frequency['purchase_count'] / total
) * 100

# ----------------------------------------------------------------------
# PLOT
# ----------------------------------------------------------------------
fig, ax = plt.subplots(figsize=(11, 6))

x = range(len(meat_frequency))
y = meat_frequency['purchase_pct']

# line
ax.plot(
    x,
    y,
    linewidth=3,
    marker='o',
    markersize=8,
    alpha=0.9
)

# area fill
ax.fill_between(
    x,
    y,
    alpha=0.2
)

# weekend shading
ax.axvspan(4.5, 6.5, alpha=0.08)

# data point labels – bigger now
for i, val in enumerate(y):
    ax.text(
        i,
        val + 0.5,
        f'{val:.1f}%',
        ha='center',
        fontsize=12          # increased from 10
    )

# formatting
ax.set_xticks(x)
ax.set_xticklabels(weekday_order, fontsize=13)   # bigger x‑tick labels

ax.set_ylabel('Percentage of Weekly Meat Purchases',
              fontsize=16)                       # bigger y‑axis label

ax.set_xlabel('Day of Week',
               fontsize=13.5)
ax.set_title(
    'Meat and Poultry Purchases as a Percentage of Weekly Total',
    fontsize=20,
    weight='bold'
)

# make y‑axis tick numbers larger
ax.tick_params(axis='y', labelsize=13)           # bigger y‑scale numbers

# give the highest label breathing room
ax.set_ylim(0, y.max() * 1.15)                  # <-- added headroom

# remove clutter
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

ax.grid(axis='y', linestyle='--', alpha=0.3)

plt.tight_layout()
plt.show()

# ----------------------------------------------------------------------
# 8. VISUALISATION – Line + Area (orange, corrected title)
# ----------------------------------------------------------------------
fig, ax = plt.subplots(figsize=(11, 6))

x = range(len(dairy_frequency))
y = dairy_frequency['purchase_pct']

# line (orange)
ax.plot(
    x,
    y,
    linewidth=3,
    marker='o',
    markersize=8,
    alpha=0.9,
    color='orange'
)

# area fill (orange, with transparency)
ax.fill_between(
    x,
    y,
    alpha=0.2,
    color='orange'
)

# weekend shading (Friday evening through Sunday)
ax.axvspan(4.5, 6.5, alpha=0.08, color='orange')

# data point labels
for i, val in enumerate(y):
    ax.text(
        i,
        val + 0.5,
        f'{val:.1f}%',
        ha='center',
        fontsize=12
    )

# formatting
ax.set_xticks(x)
ax.set_xticklabels(weekday_order, fontsize=13)

ax.set_ylabel('Percentage of Weekly Dairy Purchases',
              fontsize=16)

ax.set_xlabel('Day of Week',
              fontsize=13)

# UPDATED TITLE – reflects percentage, not raw count
ax.set_title(
    'Dairy Purchases as a Percentage of Weekly Total',
    fontsize=20,
    weight='bold'
)

# make y‑axis tick numbers larger
ax.tick_params(axis='y', labelsize=13)

# give the highest label breathing room
ax.set_ylim(0, y.max() * 1.15)

# remove clutter
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

ax.grid(axis='y', linestyle='--', alpha=0.3)

plt.tight_layout()
plt.show()

import pandas as pd
import matplotlib.pyplot as plt

# ======================================================================
# 1. PREPARE DAIRY DATA
# ======================================================================
dairy_df = df[df['category'] == 'Dairy'].copy()
dairy_df['time_stamp'] = pd.to_datetime(dairy_df['time_stamp'])
dairy_df['day_of_week'] = dairy_df['time_stamp'].dt.day_name()

weekday_order = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']

dairy_freq = (
    dairy_df.groupby('day_of_week')
    .size()
    .reindex(weekday_order, fill_value=0)
    .reset_index(name='purchase_count')
)
dairy_total = dairy_freq['purchase_count'].sum()
dairy_freq['purchase_pct'] = (dairy_freq['purchase_count'] / dairy_total) * 100

# ======================================================================
# 2. PREPARE MEAT & POULTRY DATA
# ======================================================================
meat_df = df[df['category'] == 'Meat & Poultry'].copy()
meat_df['time_stamp'] = pd.to_datetime(meat_df['time_stamp'])
meat_df['day_of_week'] = meat_df['time_stamp'].dt.day_name()

meat_freq = (
    meat_df.groupby('day_of_week')
    .size()
    .reindex(weekday_order, fill_value=0)
    .reset_index(name='purchase_count')
)
meat_total = meat_freq['purchase_count'].sum()
meat_freq['purchase_pct'] = (meat_freq['purchase_count'] / meat_total) * 100

# ======================================================================
# 3. COMBINED PLOT (no shading, no data labels)
# ======================================================================
fig, ax = plt.subplots(figsize=(12, 7))

x = range(len(weekday_order))

# --- Dairy (orange) ---
dairy_y = dairy_freq['purchase_pct']
ax.plot(x, dairy_y, linewidth=3, marker='o', markersize=8,
        alpha=0.9, color='orange', label='Dairy')
ax.fill_between(x, dairy_y, alpha=0.15, color='orange')

# --- Meat & Poultry (blue) ---
meat_y = meat_freq['purchase_pct']
ax.plot(x, meat_y, linewidth=3, marker='s', markersize=8,
        alpha=0.9, color='#1f77b4', label='Meat & Poultry')
ax.fill_between(x, meat_y, alpha=0.15, color='#1f77b4')

# axis formatting
ax.set_xticks(x)
ax.set_xticklabels(weekday_order, fontsize=13)
ax.set_ylabel('Percentage of Weekly Purchases (%)', fontsize=14)
ax.set_xlabel('Day of Week', fontsize=13)
ax.set_title('Weekly Rhythm of Dairy vs Meat & Poultry Purchases',
             fontsize=20, weight='bold')
ax.tick_params(axis='y', labelsize=13)

# headroom: take the max of both lines
max_val = max(dairy_y.max(), meat_y.max())
ax.set_ylim(0, max_val * 1.15)   # slightly reduced since no labels on top

# clean spines & grid
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.grid(axis='y', linestyle='--', alpha=0.3)

# legend
ax.legend(fontsize=12, frameon=False)

plt.tight_layout()
plt.show()

# Ensure time_stamp is datetime
df['time_stamp'] = pd.to_datetime(df['time_stamp'])

# Extract day of week (Monday=0, Sunday=6)
df['day_of_week'] = df['time_stamp'].dt.dayofweek

# Filter only Saturdays (dayofweek == 5)
saturdays = df[df['day_of_week'] == 5].copy()

# Aggregate total spend by category on Saturdays (price * quantity)
sat_spend = (
    saturdays.groupby('category')
    .apply(lambda group: (group['product_price'] * group['product_quantity']).sum())
    .sort_values(ascending=False)
    .reset_index()
)
sat_spend.columns = ['category', 'total_spend']

# Plot – horizontal bar, EXCLUDING the top spender
fig, ax = plt.subplots(figsize=(12, 8))

# Use all rows EXCEPT the first (index 0)
plot_data = sat_spend.iloc[1:]                # <-- offset by 1 to drop top value

bars = ax.barh(plot_data['category'], plot_data['total_spend'],
               color='#55A868', alpha=0.9)
ax.set_title('Distribution of Spending across Shopping Categories',
             fontsize=15, weight='bold')
ax.set_xlabel('Total Spend (£)', fontsize=16)
ax.set_ylabel('Item Category', fontsize=16)
ax.invert_yaxis()  # highest remaining spend at top

# Increase font size of axis tick labels
ax.tick_params(axis='y', labelsize=16)
ax.tick_params(axis='x', labelsize=16)
plt.tight_layout()
plt.show()

# Print the full table (including the top spender)
print(sat_spend)

# ======================================================================
# 10. VERTICAL STACKED BAR CHART – BIG, BOLD, WHITE LABELS
# ======================================================================
fig, ax = plt.subplots(figsize=(25, 24))          # larger canvas

bottom = 0
bar_width = 0.9                                   # thicker bar

for _, row in behaviour_spend.iterrows():
    group = row['behaviour_group']
    pct   = row['pct']
    color = group_colors.get(group, '#BDBDBD')

    # Draw the segment
    ax.bar(
        x=0,
        height=pct,
        width=bar_width,
        bottom=bottom,
        color=color,
        edgecolor='white',
        linewidth=2
    )

    # White label inside every segment (skip only very tiny slivers)
    if pct >= 2.5:
        label_text = f"{group}\n{pct:.1f}%"
        ax.text(
            0,
            bottom + pct / 2,
            label_text,
            ha='center',
            va='center',
            fontsize=26.3,                 # much larger
            fontweight='bold',
            color='white'                # always white
        )

    bottom += pct

# ----------------------------------------------------------------------
# 11. LEGEND – top right, outside
# ----------------------------------------------------------------------
import matplotlib.patches as mpatches

legend_patches = [
    mpatches.Patch(color='#2E7D32', label='Healthy'),
    mpatches.Patch(color='#9CCC65', label='Moderately Healthy'),
    mpatches.Patch(color='#FFCA28', label='Balanced / Mixed'),
    mpatches.Patch(color='#FF7043', label='Fairly Unhealthy'),
    mpatches.Patch(color='#C62828', label='Generally Unhealthy')
]

ax.legend(
    handles=legend_patches,
    loc='upper left',
    bbox_to_anchor=(1.02, 1.02),
    fontsize=23.5,
    title='Health Spectrum',
    title_fontsize=25,
    frameon=False
)

# ----------------------------------------------------------------------
# 12. STYLING – larger fonts everywhere
# ----------------------------------------------------------------------
ax.set_xlim(-0.5, 0.5)
ax.set_ylim(0, 100)
ax.set_xticks([])
ax.set_ylabel('Percentage of Total Food Spend (%)', fontsize=31, labelpad=15)

ax.set_title(
    'Composition of Food Shopping Spend',
    fontsize=33,
    weight='bold',
    pad=25
)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['bottom'].set_visible(False)
ax.tick_params(axis='y', labelsize=27)
ax.yaxis.grid(True, linestyle='--', alpha=0.3)

plt.tight_layout()
plt.show()
fig.savefig('behavioural_food_profile5.png', dpi=300, bbox_inches='tight')
plt.show()

# ======================================================================
# SATURDAY COOKING SPEND – EXPANDED, X LABELS UPDATED
# ======================================================================

import pandas as pd
import matplotlib.pyplot as plt

# Ensure datetime and day names are ready
df['time_stamp'] = pd.to_datetime(df['time_stamp'])
df['day_name'] = df['time_stamp'].dt.day_name()

# Filter to Saturdays and remove "Other" category
sat_df = df[(df['day_name'] == 'Saturday') & (df['category'] != 'Other')].copy()

# ------------------------------------------------------------------
# DEFINE COOKING-ORIENTED CATEGORIES
# ------------------------------------------------------------------
COOKING_CATEGORIES = [
    'Meat & Poultry',
    'Vegetables & Salad',
    'Fish & Seafood',
    'Pasta & Grains',
    'Sauces & Condiments',
    'Baking Ingredients',
    'Dairy'                # cheese, cream, butter, milk – all used in cooking
]

# ------------------------------------------------------------------
# CALCULATE TRUE SPEND (price × quantity)
# ------------------------------------------------------------------
sat_target_spend = (
    sat_df[sat_df['category'].isin(COOKING_CATEGORIES)]
    .apply(lambda row: row['product_price'] * row['product_quantity'], axis=1)
    .sum()
)
sat_other_spend = (
    sat_df[~sat_df['category'].isin(COOKING_CATEGORIES)]
    .apply(lambda row: row['product_price'] * row['product_quantity'], axis=1)
    .sum()
)

sat_total = sat_target_spend + sat_other_spend
sat_target_pct = (sat_target_spend / sat_total) * 100

# ------------------------------------------------------------------
# BEAUTIFUL CHART
# ------------------------------------------------------------------
plt.style.use('seaborn-v0_8-white')

cooking_color = '#FF6F61'
other_grey    = '#CFD8DC'

fig, ax = plt.subplots(figsize=(10, 7))

categories = ['Cooking Items', 'Non-Cooking Items']     # <--- CHANGED
pct_values = [sat_target_pct, 100 - sat_target_pct]

bars = ax.bar(categories, pct_values,
              color=[cooking_color, other_grey],
              edgecolor='white', linewidth=1.2)

for bar, pct in zip(bars, pct_values):
    ax.text(bar.get_x() + bar.get_width()/2,
            bar.get_height() + 1,
            f'{pct:.1f}%',
            ha='center', fontweight='bold', fontsize=14,
            color='#333333')

ax.set_title('Saturday Spending – Cooking vs. Non-Cooking', fontsize=20, weight='bold', pad=15)
ax.set_ylabel('Percentage of Saturday Spend (%)', fontsize=16)
ax.set_ylim(0, max(pct_values) * 1.2)
ax.spines[['top','right','left','bottom']].set_visible(False)
ax.tick_params(left=False, labelleft=True, labelsize=13)
ax.tick_params(axis='x', labelsize=14)
ax.yaxis.set_ticks_position('none')

plt.tight_layout()
plt.show()

# Print numbers
print(f"Cooking Items Spend (Sat): £{sat_target_spend:,.2f} ({sat_target_pct:.1f}%)")
print(f"Non-Cooking Items Spend (Sat): £{sat_other_spend:,.2f} ({100-sat_target_pct:.1f}%)")

import pandas as pd
import matplotlib.pyplot as plt

# Ensure time_stamp is datetime
tesco_data['time_stamp'] = pd.to_datetime(tesco_data['time_stamp'])

# Filter to Saturdays only
saturdays = tesco_data[tesco_data['time_stamp'].dt.day_name() == 'Saturday'].copy()

# Pivot table: sum of (product_price * product_quantity) per store_id on Saturdays
pivot_price = (
    saturdays.groupby('store_id', as_index=False)
    .agg(total_price=('product_price', lambda x: (x * saturdays.loc[x.index, 'product_quantity']).sum()))
)

# Sort and take top 10
top10_price = pivot_price.nlargest(10, 'total_price')

# Sort ascending for horizontal bar (largest at the top)
top10_price = top10_price.sort_values('total_price', ascending=True)

# --- Colour palette ---
colors = ['#1A237E', '#283593', '#1565C0', '#42A5F5', '#00897B',
          '#00ACC1', '#FF6F61', '#C44E52', '#5E35B1', '#FFA726']

# Plot – horizontal bars (store_id as string to avoid numeric coordinates)
fig, ax = plt.subplots(figsize=(12, 7))
bars = ax.barh(top10_price['store_id'].astype(str),   # convert to string
               top10_price['total_price'],
               color=colors[:len(top10_price)],
               edgecolor='white', linewidth=1.2)

# Add spend labels
for bar, val in zip(bars, top10_price['total_price']):
    ax.text(bar.get_width() + max(top10_price['total_price'])*0.01,
            bar.get_y() + bar.get_height()/2,
            f'£{val:,.0f}', va='center', fontsize=14, fontweight='bold')

# Styling
ax.set_title('Money Spent at Different Tesco Stores on Saturdays', fontsize=16, weight='bold', pad=15)
ax.set_xlabel('Total Spend (£)', fontsize=14)
ax.set_ylabel('Tesco Store', fontsize=16)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.xaxis.grid(True, linestyle='--', alpha=0.4)
ax.set_axisbelow(True)
ax.set_xlim(0, max(top10_price['total_price']) * 1.2)
ax.tick_params(axis='y', labelsize=16)

plt.tight_layout()
plt.show()
