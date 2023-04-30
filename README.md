Download Link: https://assignmentchef.com/product/solved-mini-project-predicting-distribution-of-a-species
<br>
<strong>library</strong>(ggplot2); <strong>theme_set</strong>(<strong>theme_light</strong>()) <strong>library</strong>(dplyr) <strong>library</strong>(readr)

<h1>Introduction</h1>

The goal of the mini-project is to predict the presence of a species (such as a tree/shrub/etc) from presence-only observations. This is useful when trying to mangage populations of species, reducing impacts of invasive species, and preserving endangered species. Predicting habitats in which it is present and grows quickly is important for managing it effectively.

The goal is to predict the distribution from <em><sub>presence-only </sub></em>data. This is a machine learning problem but it is different from the standard classification/regression setting. It is an example of a presence-only dataset. An example of what actual presence data looks like can be seen on <a href="https://www.eddmaps.org/distribution/viewmap.cfm?sub=3010">EDDMapS.</a> The data generally consists of <em>positive </em>samples only. Most observations are reported by volunteers. People generally do not report when they do not observe the plant and therefore there are no (or very few) negative samples.

Presence-only data is more common in machine learning than you may expect. Positive samples are often easier to collect than negative ones. Some obvious settings that are similar are predicting the presence of insects, pests, and diseases. A lot of businesses have access to positive samples (their customers) and have little access to negative samples (people who are not their customers). Models that deal with presence-only data are also common in other areas of machine learning, such as natural language modeling and (inverse) reinforcement learning.

In this project, we will use New England’s landscape. It is divided into rectangular cells <em><sub>L</sub></em>. Each cell is identified by a unique <sub>cellid</sub>, and for each cell we know its latitude and longitude. The training data is the number of observations <em><sub>nl </sub></em>reported that have been reported for each cell <em><sub>l ∈L</sub></em>. It is important to note that there may be significant population in a cell, but no-one has reported an observation from there. There are 19 biologically-relevant predictors available from <a href="https://worldclim.org/">WorldClim</a> which can be used to generalize the observation from the current cells.

The goal if this project is to predict observed populations in cells that are not a part of the training set. Biologists instead usually care about predicting the population in a cell, or its suitability for the species, but ground truth data on that is difficult to gather. The project has two following similar sub-problems.

<ol>

 <li>A synthetic subproblem. The population in each cell is computed from a simulated model which is based on a real plant species. The synthetic problem makes is possible to have ground truth data and describe exactly the process that generates the observations. The observations in this synthetic model are based only on a subset of the 19 predictors; there are no other considerations.</li>

 <li>A real subproblem. This is a real dataset derived from EDDMaps for a mystery plant species in New England. There may be no perfect fit for this model. Other features in addition to the ones provided may be useful when fitting the model. Spatial considerations, such as the presence of the species in nearby-cells, may also play a role in this part of the project.</li>

</ol>

<h1>Objective and evaluation</h1>

The goal is to predict, for each cell in the landscape not included in the training set, number of observations in the test set. The output should be a <em><sub>number </sub></em>for each cell not included in the training set.

<h1>Datasets</h1>

The following files are available:

<ol>

 <li><sub>csv</sub>: Landscape data: cellid, latitude, longitude, bio-features</li>

 <li><sub>csv</sub>: Presence-only data for the synthetic problem. It contains cell id, and the number of reports (<sub>freq</sub>)</li>

 <li><sub>csv</sub>: Presence-only data for the real problem. It contains cell id, and the number of reports (<sub>freq</sub>)</li>

</ol>

The test sets are, of course, secret for now.

<h2>Landscape</h2>

The landscape dataset contains the list of all cells, their coordinates, and the biologically-relevant variables. The dataset is available as the <sub>landscape.csv </sub>file:

landscape &lt;- <strong>read_csv</strong>(“landscape.csv”, col_types = <strong>cols</strong>(cellid = <strong>col_integer</strong>())) limits &lt;- <strong>function</strong>(x){<strong>c</strong>(<strong>min</strong>(x), <strong>max</strong>(x))} limits_lat &lt;- <strong>limits</strong>(landscape<strong>$</strong>lat) limits_long &lt;- <strong>limits</strong>(landscape<strong>$</strong>long)

If you would like to see how the landscape data was generated, please see the <sub>landscape.csv </sub>file.

Description of the variables is at: <a href="http://worldclim.org/bioclim">http://worldclim.org/bioclim.</a> Briefly, their meaning is a follows.

<table width="438">

 <tbody>

  <tr>

   <td width="60">Feature</td>

   <td width="379">Description</td>

  </tr>

  <tr>

   <td width="60">BIO1</td>

   <td width="379">Annual Mean Temperature</td>

  </tr>

  <tr>

   <td width="60">BIO2</td>

   <td width="379">Mean Diurnal Range (Mean of monthly (max temp – min temp))</td>

  </tr>

  <tr>

   <td width="60">BIO3</td>

   <td width="379">Isothermality (BIO2/BIO7) (* 100)</td>

  </tr>

  <tr>

   <td width="60">BIO4</td>

   <td width="379">Temperature Seasonality (standard deviation *100)</td>

  </tr>

  <tr>

   <td width="60">BIO5</td>

   <td width="379">Max Temperature of Warmest Month</td>

  </tr>

  <tr>

   <td width="60">BIO6</td>

   <td width="379">Min Temperature of Coldest Month</td>

  </tr>

  <tr>

   <td width="60">BIO7</td>

   <td width="379">Temperature Annual Range (BIO5-BIO6)</td>

  </tr>

  <tr>

   <td width="60">BIO8</td>

   <td width="379">Mean Temperature of Wettest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO9</td>

   <td width="379">Mean Temperature of Driest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO10</td>

   <td width="379">Mean Temperature of Warmest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO11</td>

   <td width="379">Mean Temperature of Coldest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO12</td>

   <td width="379">Annual Precipitation</td>

  </tr>

  <tr>

   <td width="60">BIO13</td>

   <td width="379">Precipitation of Wettest Month</td>

  </tr>

  <tr>

   <td width="60">BIO14</td>

   <td width="379">Precipitation of Driest Month</td>

  </tr>

  <tr>

   <td width="60">BIO15</td>

   <td width="379">Precipitation Seasonality (Coefficient of Variation)</td>

  </tr>

  <tr>

   <td width="60">BIO16</td>

   <td width="379">Precipitation of Wettest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO17</td>

   <td width="379">Precipitation of Driest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO18</td>

   <td width="379">Precipitation of Warmest Quarter</td>

  </tr>

  <tr>

   <td width="60">BIO19</td>

   <td width="379">Precipitation of Coldest Quarter</td>

  </tr>

 </tbody>

</table>

Lets plot the landscape data to get some sense of what it looks like. This code plots some of the climatic variables and the outline of the states. This is the variable <sub>bio13</sub>:

<table width="632">

 <tbody>

  <tr>

   <td width="632">states &lt;- <strong>map_data</strong>(“state”, regions = “.”, xlim = limits_long, ylim = limits_lat, lforce = “e”)<strong>ggplot</strong>(landscape, <strong>aes</strong>(x = long, y = lat)) <strong>+ </strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = bio13)) <strong>+ </strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states,</td>

  </tr>

  <tr>

   <td width="632">fill = NA, color = “white”) <strong>+</strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>scale_alpha_continuous</strong>(range=<strong>c</strong>(0.2,0.8)) <strong>+ </strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Precip”)</td>

  </tr>

 </tbody>

</table>

And this is <sub>bio6</sub>:

<table width="632">

 <tbody>

  <tr>

   <td width="632"><strong>ggplot</strong>(landscape, <strong>aes</strong>(x = long, y = lat)) <strong>+ </strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = bio6)) <strong>+ </strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “white”) <strong>+</strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>scale_alpha_continuous</strong>(range=<strong>c</strong>(0.2,0.8)) <strong>+ </strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Temp”)</td>

  </tr>

 </tbody>

</table>




<h2>Synthetic (Fake) Presence Data</h2>

<table width="632">

 <tbody>

  <tr>

   <td width="632">syn_feature_matrix &lt;- <strong>model.matrix</strong>(<strong>~</strong>bio2<strong>+</strong>bio3<strong>+</strong>bio5<strong>+</strong>bio8<strong>+</strong>bio16, landscape) beta &lt;- <strong>c</strong>(<strong>–</strong>2,1,0.1,<strong>–</strong>0.5,0.1,0.00001) <em># made up beta!!</em>prevalence &lt;- <strong>exp</strong>(syn_feature_matrix <strong>%*% </strong>beta) synthetic_true &lt;- <strong>data.frame</strong>(cellid = landscape<strong>$</strong>cellid, prevalence = prevalence) <strong>ggplot</strong>(synthetic_true <strong>%&gt;% </strong><strong>full_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,long,lat), by = “cellid”), <strong>aes</strong>(x = long, y = lat)) <strong>+</strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = prevalence)) <strong>+</strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “white”) <strong>+ </strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Prevalence”)</td>

  </tr>

 </tbody>

</table>

The code in this section demonstrates how the synthetic data is generated. The underlying presence data generated by this code is <em><sub>different </sub></em>from the data that is provided. The purpose of this code is to give you some information that you can use to design your algorithms.

The synthetic presence data uses only features <sub>bio2</sub>, <sub>bio3</sub>, <sub>bio5</sub>, <sub>bio8 </sub>and <sub>bio16</sub>. The dependence on these features may be wither linear or non-linear. These features are used to generate true <em><sub>prevalence </sub></em>of the species in the landscape for each. Think of this as the number of all specimen for each cell.

The synthetic observation data is sampled based on the prevalence of the species in each cell. The procedure corresponds to sampling each specimen with some small fixed probability. That mean that cells that have higher prevalence are likely to have more observations. This a common assumption with species models. We can plot the individual observations (since the observations are on top of each other, they are “jittered”).

<table width="632">

 <tbody>

  <tr>

   <td width="632"><strong>set.seed</strong>(1000) synthetic_observations &lt;- <strong>data.frame</strong>( cellid = <strong>sample</strong>(synthetic_true<strong>$</strong>cellid, 8000, replace = TRUE, prob = synthetic_true<strong>$</strong>prevalence))<strong>ggplot</strong>(synthetic_true <strong>%&gt;% </strong><strong>full_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,long,lat), by = “cellid”), <strong>aes</strong>(x = long, y = lat)) <strong>+</strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = prevalence)) <strong>+ </strong><strong>geom_jitter</strong>(data = synthetic_observations <strong>%&gt;%</strong><strong>inner_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,long,lat), by = “cellid”),size = 0.3, alpha = 0.3, color = “red”) <strong>+</strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “white”) <strong>+ </strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Prevalence”)</td>

  </tr>

 </tbody>

</table>

The results are aggregated by the cell to make it more convenient working with them.

<table width="632">

 <tbody>

  <tr>

   <td width="632">synthetic_all &lt;- synthetic_observations <strong>%&gt;% </strong><strong>group_by</strong>(cellid) <strong>%&gt;% </strong><strong>summarize</strong>(freq = <strong>n</strong>()) <strong>ggplot</strong>(synthetic_all <strong>%&gt;% </strong><strong>inner_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,lat,long), by=”cellid”), <strong>aes</strong>(x = long, y = lat)) <strong>+</strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = freq)) <strong>+</strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “blue”) <strong>+ </strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Count”)</td>

  </tr>

 </tbody>

</table>

7

6

5

4

3

2

1

Finally, about half of the observed cell are included in the training data (provided). The other half is retained for the test set (unavailable).

<table width="632">

 <tbody>

  <tr>

   <td width="632">synthetic_train &lt;- <strong>sample_frac</strong>(synthetic_all, 0.5) synthetic_test &lt;- <strong>anti_join</strong>(synthetic_all, synthetic_train, by=”cellid”) <strong>ggplot</strong>(synthetic_train <strong>%&gt;% </strong><strong>inner_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,lat,long), by=”cellid”), <strong>aes</strong>(x = long, y = lat)) <strong>+</strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = freq)) <strong>+</strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “blue”) <strong>+ </strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Count”, title=”Training”)</td>

  </tr>

 </tbody>

</table>




Training

7

6

5

4

3

2

1

<h2>Real Reported Presence Data</h2>

The real dataset is generated similarly. We take the reported observations, determine which cell they it should be included in, and then aggregate the counts by cell.

<table width="632">

 <tbody>

  <tr>

   <td width="632">real_train &lt;- <strong>read_csv</strong>(“real_train.csv”, col_types = <strong>cols</strong>(cellid = <strong>col_integer</strong>())) <strong>ggplot</strong>(real_train <strong>%&gt;% </strong><strong>inner_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid,lat,long), by=”cellid”), <strong>aes</strong>(x = long, y = lat)) <strong>+</strong><strong>geom_raster</strong>(<strong>aes</strong>(fill = freq)) <strong>+</strong><strong>geom_polygon</strong>(<strong>aes</strong>(group = group), data = states, fill = NA, color = “blue”) <strong>+ </strong><strong>scale_fill_viridis_c</strong>() <strong>+</strong><strong>coord_fixed</strong>() <strong>+ </strong><strong>labs</strong>(x = “Longitude”, y = “Latitude”, fill = “Count”, title=”Training”)</td>

  </tr>

 </tbody>

</table>

Training

<h1>Submission and Evaluation</h1>

The goal is to predicted the number of observations for each cell in the landscape that matches the test as closely as possible. <strong>The evaluation metric is as follows. </strong>Each correctly-predicted observation <strong>earns 1 point</strong>, while each incorrectly-predicted observation <strong><sub>loses 0.3 </sub></strong>points. For examples, lets say that cell 1 in the test set has 5 observations. If you predict 3 then you gain 3 points, and if you predict 7 you lose 0.6 (2*0.3) points. The predicted counts can be real numbers, not just integers.

You should submit the following files:

<ol>

 <li>A very <em><sub>short </sub></em>document (Rmd, Jupyter, Latex, PDF) describing the method(s) used and results (1-2 pages including plots)</li>

 <li>csv CSV file with two columns: cellid and pred_freq for cells that are NOT in the training set synthetic_train.csv</li>

 <li>csv: CSV file with two columns: cellid and pred_freq for cells that are NOT in the training set real_train.csv</li>

</ol>

<h2>Observations in Test Sets</h2>

The number of observations in the test sets are:

<ul>

 <li>Synthetic: 5013</li>

 <li>Real: 1981</li>

</ul>

<h2>Example Evaluation</h2>

Lets say I figure out the precise coefficients <em><sub>β </sub></em>for the synthetic problem. I can then generate predictions directly from the estimated prevalence data. I also need to remove all cells that are in the training data from the prediction.

<table width="632">

 <tbody>

  <tr>

   <td width="632">predicted &lt;- <strong>data.frame</strong>( cellid = <strong>sample</strong>(synthetic_true<strong>$</strong>cellid, 3000, replace = TRUE, prob = synthetic_true<strong>$</strong>prevalence))predicted &lt;- predicted <strong>%&gt;% </strong><strong>group_by</strong>(cellid) <strong>%&gt;% </strong><strong>summarize</strong>(pred_freq = <strong>n</strong>()) predicted &lt;- <strong>anti_join</strong>(predicted, synthetic_train, by = “cellid”)</td>

  </tr>

 </tbody>

</table>

The score can now be computed by joining the results with the test set and computing the difference. The statement below also filters out any extraneous columns, cells that are not in the landscape, and cells from the training set. It assumes that all missing predictions are 0.

<table width="632">

 <tbody>

  <tr>

   <td width="632">combined &lt;predicted <strong>%&gt;% </strong><strong>select</strong>(cellid, pred_freq) <strong>%&gt;% </strong><strong>anti_join</strong>(synthetic_train, by = “cellid”) <strong>%&gt;% </strong><strong>left_join</strong>(landscape <strong>%&gt;% </strong><strong>select</strong>(cellid), by=”cellid”) <strong>%&gt;% </strong><strong>full_join</strong>(synthetic_test, by=”cellid”) <strong>%&gt;% </strong>tidyr<strong>::</strong><strong>replace_na</strong>(<strong>list</strong>(pred_freq = 0, freq = 0))<strong>sum</strong>(<strong>pmin</strong>(combined<strong>$</strong>freq, combined<strong>$</strong>pred_freq)) <strong>–</strong>0.3 <strong>* </strong><strong>sum</strong>(<strong>pmax</strong>(combined<strong>$</strong>pred_freq <strong>– </strong>combined<strong>$</strong>freq, 0))</td>

  </tr>

 </tbody>

</table>

## [1] 96.7

<h1>Simple Analysis and Ideas</h1>

Do not forget about feature transformations.

You can try any technique that you like. Feel free to focus on either the real or the synthetic problem. There are many more considerations that are important when it comes to the real data sets. Some examples:

<ul>

 <li>The density of the data is subject to density of the human population.</li>

 <li>The assumption that there are more observations when the plant is more prevalent may not be true. If it is growing everywhere nobody bothers to report it.</li>

 <li>There are spatial effects due to how the plant spreads.</li>

 <li>Other features in addition to the ones provided may be useful. You are on your own where to get them. To get started, think of using some of the methods that we have covered. Can you use them in a problem like this? Perhaps you may want to think about the expressing the likelihood of this dataset and base your method on that.</li>

</ul>

If you would like to explore some of the literature that has been written on the topic, I recommend these papers as a good starting point:

<ul>

 <li>Phillips, S. J., Dudik, M., &amp; Schapire, R. E. (2004). A maximum entropy approach to species distribution modeling. In International Conference on Machine Learning (ICML).</li>

 <li>Royle, J. A., Chandler, R. B., Yackulic, C., &amp; Nichols, J. D. (2012). Likelihood analysis of species occurrence probability from presence-only data for modelling species distributions. Methods in Ecology and Evolution, 3, 545–554.</li>

 <li>Hastie, T., &amp; Fithian, W. (2013). Inference from presence-only data; the ongoing controversy. Ecography, 36(8), 864–867.</li>

 <li>Fithian, W., &amp; Hastie, T. (2013). Finite-sample equivalence in statistical models for presence-only data. Annals of Applied Statistics, 7(4), 1917–1939.</li>

 <li>Renner, I. W., &amp; Warton, D. I. (2013). Equivalence of MAXENT and Poisson Point Process Models for Species Distribution Modeling in Ecology. Biometrics, 69(1), 274–281.</li>

</ul>