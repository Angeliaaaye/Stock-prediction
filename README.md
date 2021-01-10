# Stock-prediction
1851728 数据挖掘大作业

数据处理、特征工程：
1、delta_perday。该特征的计算方法为当天Highest与Lowest的差值，指当天股价的最高值与最低值的差值。
2、deal_per_count。该特征的计算方法为Dealamount/Dealcount，指每单成交金额的平均值。
3、isRise。若该日Percent>0，则表示股票升值，isRise=1；反之isRise=0。
4、根据月份、季度、年份聚合特征。首先使用pandas的to_period函数，从Date中提取出月份、季度、年份信息。
根据月份、年份、季度信息进行如下聚合操作：
Percent、RiseAmount：取某时间段最大、最小、平均值操作。
End、Start：取某时间段最大、最小、平均值操作。
isRise:取某段时间求和、计数操作。
5、计算出一周之内、一月之内、三月之内等等的涨幅。
以一周涨幅为例，计算出一周之内收盘价的涨幅、一周之内收盘价的上涨比例。
生成week_delta（计算一周之内的涨幅）、week_delta_percent（一周之内收盘价的上涨比例）、half_month_delta（半月之内收盘价的涨幅）、half_month_delta_percent（半月之内收盘价的上涨比例）、month_delta、month_delta_percent、three_month_delta、three_month_delta_percent、six_month_delta、six_month_delta_percent、year_delta、year_delta_percent。
6、标准化。
因为各种特征数据的规模不同，如Dealamount这一列的数据极大，而Percent这一列的值几乎在-10到10之内。许多模型对于数据的大小非常敏感，因此如果不进行标准化，模型的准确度可能会受影响。标准化公式为：(X-mean)/std  计算时对每个属性/每列分别进行。将数据按期属性（按列进行）减去其均值，并处以其方差。得到的结果是，对于每个属性/每列来说所有数据都聚集在0附近，方差为1。这种标准化成为Z-score。这一步骤使用sklearn的processing.scale函数完成。
7、生成时序特征。
股票预测需要基于之前的n天股票数据进行预测。将过去n天的数据集成到一行中，形成时序特征。代码如下：
生成的时序特征有：前1-21天的End、Highest、Lowest、Start、Lastend、RiseAmount、Percent、dealpercount、Dealta_perday特征。下图为前21天的时序特征举例。
特征工程完毕后，每行共有259个特征。进行时间一致性的特征筛选：将特征放到 LightGBM 模型中训练，训练集为第一年的数据，测试集为最后一年的数据。观察在过去了一段时间的情况下，每个特征的 trn 值是否都能超过 0.5。若有特征未超过 0.5，表示其对模型作用不大，不具有时间一致性，将其舍去。这样的筛选能够找到那些随着时间变化不那么重要了的特征。本模型发现所有特征的trn都超过0.5，因此不舍去特征。
 
模型建立部分：
我选取的模型有LightGBM regressor、SGDregressor、SVR。在确定了三个基模型之后，我们对三个模型进行stacking操作。

