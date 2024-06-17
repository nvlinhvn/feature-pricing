# feature-pricing
Dynamic pricing for launching a pushup feature

# Problem Statement
One of my client is a second hand marketplace where sellers upload items and try to sell them to buyers. It is normal that supply is larger than demand, and not all items are being sold. However, the aim is to make sellers happy. In order to provide sellers with as many opportunities to sell as possible, they can buy specific features to promote their items. One of the available promotion features is the “push up” feature that boosts item visibility.
“Push ups” increase the attention given to a specific item on the platform by increasing the visibility for potential buyers (i.e. more views on the homepage and category page) for 3 days after having purchased the “push up” feature. As a result, the item that is up for sale will get significantly more views (on average). Currently, it costs €2 to buy the “push up” feature. The client is considering whether this is the best monetization strategy for the value added services.

# Basic Analysis

![Feature Adoption](https://raw.githubusercontent.com/nvlinhvn/feature-pricing/linh-dev/img/top_30_feature_adoption.png)
* We can see that the top categories generating the most revenue are `FOOTWEAR_W_TRAINERS`, `WOMENS_DRESSES`, and `WOMENS_TOPS_T_SHIRTS`. This suggests that the "push up" feature is performing well in these categories.
* ~60% of revenue from push ups are coming from top 20 categories (10% of total categories)
* Some categories have a higher revenue per listing compared to others. This indicates potential for optimizing the pricing strategy based on category-specific factors. There’re miss opportunities where some categories like `GIR_SETS`, `CLO_SETS` (undefined category 3) have a high number of listings (> 1000) but no revenue from pushups
* This suggests that there might be room for improvement related to pricing models and market/category dynamics. For example, it might be beneficial to adjust the price of the “push up” feature based on each category.


# Correlation
![Listing vs Feature Revenue](https://raw.githubusercontent.com/nvlinhvn/feature-pricing/linh-dev/img/listing_vs_feature_revenue.png)

![Price vs Feature Revenue](https://raw.githubusercontent.com/nvlinhvn/feature-pricing/linh-dev/img/price_vs_feature_revenue.png)

* Revenue of the feature are highly correlated with the category competitiveness (number of listing). The correlation is ~0.87
* For each additional listing, the revenue from push-ups is expected to increase by approximately 0.0505€
* For each additional 1€ in listing price, the revenue from push-ups is naively expected to increase by approximately 16.9€ 
* The model relationship between price and feature revenue is not the best. However, for the sake of simplicity, assume this is the relationship we can apply for dynamic pricing.

# Dynamic Pricing

Of course, keeping a fixed "push up" price throughout the platform is simple and ease for sellers, However:
* it doesn't account for category-specific factors like average listing price and competition
* May deter sellers in lower-priced categories from using the feature
* Potential loss of revenue in high-demand, high-priced categories
* No personalization
* For example, there’re miss opportunities where some categories like `GIR_SETS`, `CLO_SETS` (undefined category 3) have a high number of listings (> 1000) but no revenue from
pushups.
* Less adaptive to market dynamics. For example, there's a sudden surge in demand for the “Orange-Sport” clothes thanks to the Netherlands playing in Euro 2024. Sellers would like to pay more than 2€ to promote their listings on the trend. A fixed pricing model may not be the best case here.

Hence, an alternative pricing strategy could be a dynamic pricing model based on:
* Category-specific
* Number of listing
* Avg Listing Price

Pros:
* High Adaptability based on demand/market or seasonality
* Maximize Revenue Opportunities: charging more for high-demand categories and less for low-demand ones → fairness in term of perceived value
* More option for personalization to align with seller needs and use case
Cons:
* Higher cost of maintenance, monitoring and implementation in the system due to complex logic. Financial revenue prediction model also needs to adjust with dynamic pricing → more challenging in financial planning.
* Dynamic pricing is a black box for sellers. It can lead to mistrust and confusion when sellers do not fully understand, and no transparency.

For example, the very naive dynamic average listing price of each category:

* Categories with avg. listing price < €10: "Push up" price of €1
* Categories with avg. listing price €10-€50: "Push up" price of €2
* Categories with avg. listing price > €50: "Push up" price of €3

## Modeling
The alternative of fixed pricing is dynamic pricing. The dynamic pricing can be based on Category-specific. The weights of Category-specific can be defined based on `Number of listing`, and `Avg Listing Price`

* Keep in mind that for dynamic pricing, we should cap the maximum (willingness to pay from seller)
* The maximum cap per category can be defined as `base_price / avg_listing_price_eur` (%) (`base_price` = 2)
* The `number of listings` in a category are positively correlated to competitiveness and demand. A higher number of listings indicates higher competition for visibility, potentially increasing the value of the "push up" feature. So, high `number of listing` category should have higher weights.
* Also, `avg listing price` reflects the overall value category. Categories with higher average listing prices may suggest higher-end product, where sellers may be willing to pay more to promote their listings.
* However, based on the data, `number of listings` have high correlation with revenue while `avg listing price` doesn't. * Hence, the weight of each category can be defined as `number of listings / avg(number of listings) - 1`.
     - If weights is negative, a category is sort of under-competitive average market --> we charge lower than base price
     - Else, a category is sort of over-competitive w.r.t average market --> we charge higher base price
* "Higher" or "Lower" value in this context, therefore, reflects with the market average in the platform
* In order to estimate new revenue after adjusting pricing, and based on the linear relationship derived above, it's fair to assume the revenue change is proportional to the change in price with coeff = 16.9

We can redistribute the feature price by each category, as can be seen:

![Feature New Price](https://raw.githubusercontent.com/nvlinhvn/feature-pricing/linh-dev/img/Feature New Price.png)

* Now, >75% of category now can have cheaper pushup price than 2 euro. This means more opportunities for sellers to use feature when price is cheaper while there's no significant change (confirmed with t-test) in feature revenue stream

