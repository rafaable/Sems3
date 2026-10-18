Highlights

    •
    An oyster freshness classification method based on E-nose is proposed.
    •
    The method combines E-nose with traditional pysicochemical detection.
    •
    A PriorBoost-CNN-LSTM model is developed to enhance data analysis performance.
    •
    The accuracy of fine-tuned freshness prediction is boosted from 69.7 % to 81.2 %.
    •
    Hourly freshness estimation is validated under varying temperature conditions.

Abstract
Oysters are highly nutritious, but due to their high moisture and protein content, they are particularly prone to spoilage under unstable temperature conditions, posing significant challenges for food safety and supply chain management. Typical freshness detection methods widely used now show accurate result, but are destructive, time-consuming and costly, such as total viable count (TVC), total volatile basic - nitrogen (TVB-N), gas chromatography - mass spectrometry (GC-MS), and biogenic amines (BAs) testing. To address this issue, an oyster freshness detection method is proposed based on electronic nose (E-nose) technology combining with a novel freshness classification model named PriorBoost-CNN-LSTM. The relationship between E-nose sensor responses and typical physicochemical detection results was explored under varying temperature conditions and high related prior information of TVC, TVB-N, BAs with gas sensors were incorporated to the model to enhance the performance of E-nose system. This model is capable of predicting subtle freshness variations on an hourly basis under temperatures of 4 °C, 12 °C, 20 °C, and 28 °C, with accuracy rates increasing from 69.7 % to 81.2 % compared to the unboosted model. Experimental results verified a more efficient and precise solution for oyster shelf life management and the approach can be a good reference for other food’s freshness evaluation.
Keywords
Oyster storage
; E-nose
; Freshness detection
; PriorBoost-CNN-LSTM model
; Shelf life management
; Data fusion analysis

    Previous article in this issue
    Next article in this issue

1. Introduction
With the continuous growth of the global seafood market, the demand for freshness detection of aquatic products has been steadily increasing. Oysters, prized for their rich flavor and high nutritional value, are widely favored (Wright et al., 2018, Tan et al., 2024) and play a significant role in China aquaculture industry, with production ranking among the top globally (Botta et al., 2020, Peng et al., 2021). The freshness of oysters not only directly determines their market value but also impacts consumer health (Biji et al., 2016, Jaguey-Hernandez et al., 2021, Malle and Poumeyrol, 1989). However, due to issues such as discoloration (Pérez-Mateos et al., 2002), spoilage (Filippini et al., 2023, Zhang et al., 2021), and microbial contamination within the scope of food adulteration (Arslan et al., 2023, Bansal et al., 2017), accurately predicting their freshness remains a major challenge for the market. Although typical and traditional detection methods, such as sensory evaluation, total volatile basic nitrogen (TVB-N), biogenic amines (BAs), and total viable count (TVC), can reflect oyster quality to some extent, they are often limited by factors such as strong subjectivity, high costs, time consumption, complex procedures, and the potential damage caused to the oysters during the detection process.
During storage, production, transportation, and sale, variations in temperature and humidity can promote microbial growth, leading to nutrient loss and the generation of harmful compounds, thereby accelerating spoilage and posing economic losses and health risks. Therefore, loads of researches have been performed focusing on studying the influencing factors on freshness of oysters. Metabolites of packaged oysters stored at different temperatures were analyzed by gas chromatography with mass spectrometry (GC-MS) equipment, and key freshness indicators were determined combining with multivariate statistical techniques (Park et al., 2024). Quality and freshness changes of shelled Pacific oysters (Crassostrea gigas) stored at -1 °C for 28 days were studied by assessing the relationship between the storage state and the biochemical properties such as microbial analysis, adenosine triphosphate (ATP)-related compounds, pH, free amino acids, and sensory evaluations, showing that the superchilling storage significantly inhibited microbial growth (Dong et al., 2023). A deep learning method combining malondialdehyde (MDA) and total sulfhydryl (SH) content with oyster meat image classification was proposed by Lu et al. (2023), achieving classification accuracies up to 92.72 % to predict MDA content, SH content, and storage days based on an oyster image dataset, offering new insights into oyster freshness prediction using computer vision and artificial intelligence (AI). A WeChat mini-program was developed to quickly assess the freshness of raw Atlantic salmon and oysters by detecting color changes in curcumin films (Chen et al., 2023). Experiments were performed for salmon and oysters stored at 4 °C for 7 days and 6 days relatively, showing results aligned well with K-value and TVB-N, indicating that both seafood began to spoil after 5–6 days, which demonstrates that the mini-program provides a rapid and reliable method for on-site freshness detection. Furthermore, neutral protease treatment has been proved improving the quality and biochemical properties of Pacific oysters under superchilling (-1.5 °C) and refrigeration (4 °C) storage state, showing high glycogen, flavor nucleotides, and sensory scores (Su et al., 2024). Superchilling at -1.5 °C slowed the increase of TVB-N, TVC, and pH, maintaining better quality than refrigeration. In summary, biological indicators including TVB-N, TVC, and pH have been proved effective to evaluate oyster freshness. However, the corresponding physicochemicl experiments to obtain the above indicators are costly, complex and destructive.
In recent years, Electronic nose (E-nose), a bionic olfactory detection technology, has been rapidly developed due to its advantages of low cost, fast response, non-destructive, and has been widely applied in agriculture (Fu et al., 2023), forestry (Wang et al., 2020), fisheries (Grassi et al., 2022), and food quality testing (Damdam et al., 2023, Huang et al., 2023, Ren et al., 2024, Surjith et al., 2024, Yu et al., 2022). The E-nose simulates the human olfactory system to detect volatile gases released by samples and uses the response signals from a sensor array for quantitative analysis (Chen et al., 2022).
However, it remains a key challenge for E-nose applications to build a reliable classification model in food freshness detection, due to the unknown volatile process and the quite complex composition and low concentration of compounds. The application of machine learning algorithms has provided expanded solutions, with commonly used algorithms including decision trees (DT), support vector machines (SVM), and artificial neural networks (ANN) (Ye et al., 2021). For instance, models with linear discriminant analysis (LDA), logistic regression (LR), SVM, and decision trees (DT) were built up based on data collected from the PEN3 E-nose with 10 sensors to classify four labeled primary aroma types of plain yogurts, which are fermented, cheesy, milky, and fruity aroma, and the SVM-based model demonstrated the best performance achieving an accuracy of 0.933 (Zeng et al., 2023). An ANN model with a topology of 8 input nodes - 15 hidden nodes - 5 output nodes was proposed to predict the shelf-life of some bread by an E-nose with 8 sensors, while the freshness-aroma of this white bread is enhanced by adding L-proline to bread dough (Bose et al., 2023). A high correlation ratio (R² = 0.9785) between predicted and experimental values was obtained with a measurement time interval of 2 days and the Mahalanobis distance method was applied to determine the bread's freshness index, which correlated with its typical formation. Convolutional Neural Networks (CNN) have been paid loads of focuses and applied widely. A CNN model was studied to detect mutton freshness using the TVB-N as the key freshness index based on an integration of PEN3 E-nose and the hyperspectral imaging (HSI) techniques (Liu et al., 2022). Seven responsive sensors were selected from a total of 10 sensors based on histogram statistics to form the E-nose sensor array. The reflectance spectra and image features were extracted from HSI images using random frog and Pearson correlation analyses. The TVB-N content of mutton was then predicted using an input modified convolutional neural network to process the measurement data with time interval of 3 days, yielding a root mean square error of 3.039 mg/100 g, a correlation coefficient of 0.920, and a performance deviation ratio of 3.59. An RGA-Net model was proposed to classify pork quality based on PEN3 E-nose system (Chang and Song, 2023). The model introduces a gas attention module to focus on critical gas features, and uses a residual dense block to prevent feature degradation in deep networks, achieving the accuracy of 97.75 %, precision of 97.91 %, and recall of 98.39 % based on a measurement time interval of 1 day, outperforming other gas information classification methods. The Neural Architecture Search Network (NasNet) and Long Short-Term Memory Networks (LSTM) approach was proposed to predict the fish freshness and demonstrated by two imbalanced datasets, achieving remarkable performance with a Matthew's correlation coefficient (MCC) and Cohen's kappa coefficient (KC) of 99.1 %, and cross-validation using a 5-fold method resulted in MCC and KC values of 97 % (Lanjewar and Panchbhai, 2024).
According to above review, it can be seen that E-nose has been proved a prospective approach to detect the freshness of food combined with the advanced pattern recognition methods, including statistical and modern artificial intelligent algorithms. Deep learning methods, such as CNNs (Zeng et al., 2025, Xiong et al., 2023), LSTMs (Bakiler and Güney, 2021, Chen et al., 2024), GNNs (Feng et al., 2024), have been adopted and showed good progress in E-nose systems. General approach to study the freshness of food is to detect the key indicators and divide the freshness level by several days. For example, the foods are labeled as Fresh and NOT-fresh based on the subjective sensory judgements from people. Even though a high accuracy of freshness evaluation can be obtained, its measuring process is settled discretely and the time interval is settled quite large, which is 1 day at least. Hence, it is still a great challenge to continuously detect the freshness of food with smaller time intervals based on E-nose. And the challenges come from the low concentration of key volatiles emitted, low sensor responses, low identity of features, and the unknown relationships between them.
To address these challenges, this study explores the response relationships between the E-nose sensor array and volatile gases, TVB-N, TVC, BAs, and other physicochemical indicators during oyster spoilage continuously and in detail. By integrating the Squeeze-and-Excitation mechanism (Hu et al., 2018), key features denoting the freshness are extracted from the gas sensor data. A model combining the Convolutional Neural Network with Long Short-Term Memory (CNN-LSTM) is then constructed to classify the freshness based on the E-nose data response to oysters in different states. The detection is continuous with a data acquisition rate of 1 Hz and a high prediction accuracy is achieved with a time interval of 1 h. Detailed experiments have been performed to verify the proposed method. Results show that the proposed method overcomes the limitations of traditional detection methods in terms of complexity and cost, offering an innovative and efficient solution for non-destructive food freshness detection under varying storage conditions and spoilage rates.
2. Materials and methods
2.1. Electronic nose (E-nose) system setup
A laboratory-developed E-nose system consisting of five main modules was employed to detect the freshness of oysters, which are a gas sensor array, a signal acquisition module, a data transmission module, a PC, and a power supply module. The gas sensor array concludes 10 metal oxide semiconductor sensors, chosen for their low cost and quick response to gases. The sensors used were TGS2602, TGS2603, TGS2612, TGS2630, MQ137, MQ135, TGS2600, TGS2610, TGS2620, and TGS2611, denoted as S1, S2, …, S10 respectively. The TGS series sensors are from Figaro Engineering (Osaka, Japan) and the MQ series sensors are from Winsen Technology (Zhengzhou, China).
The sensor array was placed inside a sealed measurement chamber with inlet and outlet. The oyster samples to be tested were placed in a bottle assembled in a high-low temperature test chamber (Shanghai Yiheng Technology, China). The odor of oysters was drawn into the sensor array chamber by a pump through a tube forming a closed loop, while a three-way valve was employed to regulate the flow of the oyster odor and clean air. Gas sensors responses to the emitted odors drawn into the measurement chamber were acquired as different voltage values continuously and transmitted to a computer via the signal acquisition and data transmission module with a frequency of 1 Hz. For detailed information on system setup and data acquisition, please refer to references (Zhang et al., 2022; Liu et al., 2023Liu et al., 2023).
2.2. Reagents and equipment
Besides the above E-nose setup, the following reagents and equipments were also employed in the following experiments.Acetonitrile (High Performance Liquid Chromatography grade) was purchased from Merck KGaA (Darmstadt, Germany). Sodium hydroxide (analytical grade) and Sodium bicarbonate (analytical grade) were purchased from Tianjin Comio (Tianjin, China). Biogenic amines mixed standard was purchased from Tianjin Alta Technology (Tianjin, China). Dansyl chloride (analytical grade) was purchased from China National Pharmaceutical Group (Shanghai, China). Perchloric acid (analytical grade), Ammonium acetate solution, Ammonium hydroxide, Magnesium oxide (MgO), and Boric acid solution were purchased from Laiyang Fine Chemical (Yantai, China).
Agilent 1100 equipped with a diode array detector and Poroshell 120 EC-C18 column was purchased from Agilent Technologies (Santa Clara, California, USA). PTFE filter membrane, Kjeldahl distillation apparatus, and computer measurement system were provided by Jienuo Testing (Yantai, China).
2.3. Oyster samples and preparation
All oyster samples were sourced from the Dongfang Ocean Oyster Farm (Yantai, China), which were fresh Pacific oysters of uniform size and sufficient quantity selected for the experiment. Considering the difficulty of monitoring volatile gases from whole oysters and in order to maintain optimal freshness while preventing further contamination during transportation, prior arrangements were made with the supplier to process the oysters by partially removing the shells. The fresh, partially shelled oysters were placed in foam boxes sealed with ice packs and promptly transported to the laboratory for further shell removal before proceeding with subsequent experiments.
After shell removal, the fresh oysters were evenly divided into four groups, each weighing approximately 250 g, and stored at 4°C, 12°C, 20°C, and 28°C, respectively, and were named Group G1, G2, G3, and G4 respectively. The odor detection experiment continued until significant spoilage was observed, characterized by a strong fishy odor, yellow discoloration, and structural changes. Since the spoilage process is closely related to the storage temperature, the duration of the experiment for each group was settled correspondingly different, as Group G1 for 10 days, Group G2 for 7 days, Group G3 for 3 days, and Group G4 for 2 days.
The four groups of oysters were prepared for the following experiments, which were the sensory evaluation, TVC analysis, TVB-N analysis, BAs analysis and E-nose detection. Detection schedules and time intervals were set as shown in Table 1, separately according to the intrinsic limitations of the detection methods.

Table 1. Duration and time intervals of 4 groups of oysters for 5 kinds of detection methods.
Detection methods	G1
(4℃, 10days)	G2
(12℃, 7days)	G3
(20℃, 7days)	G4
(28℃, 2days)	Note
Sensory evaluation	24 h	24 h	12 h	4 h	Limited human sensory resolution
TVC	48 h for first 7 days
24 h for last 3 days	24 h	12 h	4 h	Test time lasts
about 72 h
TVB-N	48 h for first 7 days
24 h for last 3 days	24 h	12 h	4 h	Test time lasts
about 1 h
BAs	48 h for first 7 days
24 h for last 3 days	24 h	12 h	4 h	Test time lasts
about 2 h
E-nose detection	About 10 mins per measuring cycle	Test time lasts
about 10 mins
TVC (total viable count); TVB-N (total volatile basic nitrogen); BAs (biogenic amines).
2.4. Sensory evaluation
Sensory evaluation is a most traditional way to assess the freshness. In addition to odor, recent studies have further proven that color assessment can serve as an effective method for detecting microbial contamination and adulteration in food (Keskin et al., 2022). Five trained evaluators were involved in carefully assessing the visual, tactile, and olfactory properties of the oysters. They recorded their observations separately for the four groups of oysters. The initial state of the oysters in all four groups was recorded having the same freshness at the beginning of the experiment. Subsequent observations were made every 24 h for Group G1 and G2, every 12 h for Group G3, and every 4 h for Group G4, as shown in Table 1. The recording intervals were set decreased from Group G1 to G4, because their spoilage rates increased greatly as the storage temperature increased, and human sensory resolution was limited.
2.5. Total viable count (TVC) analysis
The TVC detection method strictly follows the Chinese national standard "GB 4789.2–2022 National Food Safety Standard, Food Microbiological Examination, Determination of Total Viable Count" (National Health Commission of the People's Republic of China, 2022). The detection was performed with the time intervals as shown in Table 1. Each collected oyster sample was analyzed using the plate count method to determine the TVC, as following. A 5 g oyster sample was placed in a 50 mL sterile cup containing physiological saline, and homogenized at 6000 r/min for 2 min. Then, 1 mL of the homogenized solution was transferred into 9 mL of sterile physiological saline for a 10-fold gradient dilution. Three suitable dilution solutions were selected and incubated at 36°C for 72 h. The colony count was expressed as colony-forming units (CFU). The entire experiment was conducted under sterile conditions, and prior to each operation, the hands of the operator were disinfected with 75 % alcohol to prevent contamination from environmental bacteria, ensuring the accuracy of the results.
2.6. Total volatile basic nitrogen (TVB-N) analysis
The detection schedule of TVB-N for all four groups was designed with the time intervals as shown in Table 1. Each TVB-N experiment began by weighing 1 g of MgO, which was set aside for later use. The oyster samples were then homogenized using a grinder, and 10 g of the homogenized sample was weighed and placed into a distillation tube. To ensure proper dispersion, 75 mL of distilled water was added to the tube, which was thoroughly shaken and allowed to soak for 30 min. After the soaking period, 1 g of MgO was added to the tube to facilitate the distillation process. Next, the Kjeldahl distillation apparatus was set up, ensuring that the alkali solution was added in the appropriate volume, dilution water was adjusted as needed, and 30 mL of boric acid solution was used as the receiving solution. The computer measurement system was then activated, and the distillation process was initiated, running for 3 min. During this time, the TVB-N value was automatically recorded by the system and exported for further analysis. The specific experimental steps are referenced in Wang, B et al. (2024).
2.7. Biogenic amines (BAs) analysis
The detection schedule of BAs for all four groups was designed with the time intervals as shown in Table 1. Each BAs analysis began by accurately weighing the oyster samples and crushing them to ensure homogeneity. The homogenized samples were then divided into four equal portions, which were stored under open conditions. Once each portion reached the designated temperature, the timing began, and the process was documented with photographs for further analysis. For each group of oysters, two 5 g samples were taken for biogenic amine analysis. To facilitate the analysis, 200 μL of 1,7-diaminoheptane (1000 μg/mL) was added as an internal standard, followed by the addition of 25 mL of 0.40 mol/L perchloric acid solution. The mixture was vortexed for 5 min and centrifuged for 5 min. Then, 1 mL of the homogenized solution was taken and 100 μL of 2 mol/L sodium hydroxide, 2 mL of 10 mg/mL dansyl chloride solution, and 300 μL of saturated sodium bicarbonate solution were added. The mixture underwent a derivatization reaction at 60°C for 30 min. After cooling to room temperature, 100 μL of ammonia solution was added, and the volume was adjusted to 5 mL with acetonitrile. The solution was then mixed and filtered through a 0.45 μm PTFE filter membrane, followed by injection into the high-performance liquid chromatography system. Separation was performed using a Poroshell 120 EC-C18 column (250 mm × 4.6 mm, 5 μm) with a UV detector set at a wavelength of 254 nm. The injection volume was 10 μL, and the column temperature was maintained at 35°C. The mobile phases were as follows: phase A was 90 % acetonitrile / 10 % water solution containing 0.1 % acetic acid and 0.01 mol/L ammonium acetate, and phase B was 10 % acetonitrile / 90 % water solution containing 0.1 % acetic acid and 0.01 mol/L ammonium acetate. The flow rate was 1.0 mL/min. The gradient elution program was as follows: at 0 min, phase A was 60 % and phase B was 40 %. From 0–12 min, phase A increased to 85 %, and phase B decreased to 15 %. From 12–15 min, phase A increased to 100 %, and phase B decreased to 0 %. From 15–20 min, phase A was maintained at 100 %, and phase B remained at 0 %. From 20–20.1 min, phase A decreased to 60 %, and phase B increased to 40 %. From 20.1–25 min, both phases A and B were maintained at 60 % and 40 %. Subsequently, the eight biogenic amines in the oysters were subjected to qualitative and quantitative analysis to examine their concentration levels and variation trends. The internal standard method was employed to ensure the accuracy and stability of the results throughout the experiment. Specific experimental steps are referenced in Wang, B et al. (2024).
2.8. Data acquisition of E-nose responses to oyster odors
Oysters of Group G1, G2, G3 and G4 are then detected by the E-nose system. The acquisition process of E-nose responses is shown in Fig. 1, Fig. 2. All sensors in the array were preheated 12 h in advance, and then the oyster odor sample chamber was connected to the E-nose system.
Fig. 1

    Download: Download high-res image (220KB)
    Download: Download full-size image

Fig. 1. Experimental setups to acquire the E-nose responses to oyster odors.
Fig. 2

    Download: Download high-res image (109KB)
    Download: Download full-size image

Fig. 2. The continuously detection process of E-nose responses to oyster odors.
The detailed detecting process of E-nose responses to oysters were performed automatically as following. At time T1, the micro vacuum pump and electromagnetic three-way valve were turned on. Clean air was first drawn in for 20 min as a long cleaning cycle to stabilize the baseline of sensors. At time T2, the three-way valve automatically switched to the oyster odor extraction mode. The oyster odor was pumped into the sensor chamber for 300 s, and the sensor response signals were acquired and transmitted. At time T3, the three-way valve switched to extract clean air for 300 s as a short cleaning cycle, during which the sensor response was recovered to its baseline. Hence, a cycle of sensor response-recovering procedure lasted for 600 s was completed. At time T4, go back to T2 and a new sensor response-recovering cycle was started. The T2-T3 stage of 600 s was cycled 4 times. On Cycle 5, after the oyster odor was pumped in for 300 s, another 20 min long cleaning cycle was performed before entering the next 5 response-recovering cycles. The whole process continued until to final end of the experiment, hence a large amount of real time response data was obtained.
2.9. Construction methodology of the prediction model for oyster freshness
The main purpose of the study is to construct a high precision prediction model for oyster freshness based on the E-nose. Although sensory evaluation is the most direct way to evaluate the freshness and the results are quite subjective, the sensory evaluation results are general approach to decide the rough freshness levels for machine learning. Although the TVB-N, TVC, and BAs detection take long time for one measurement and they are destructive approaches, the evaluation results are accurate and reliable. Hence, a detailed study on the impact of results of sensory evaluation, TVB-N, TVC, and BAs (including β-phenylethylamine, histamine, tyramine, cadaverine, putrescine, spermidine, and spermine) on oyster freshness was firstly performed and the most related factors with the spoilage process were selected out. Then these factors were incorporated as prior knowledge and integrated into the pattern recognition model for E-nose data analysis to achieve more precise classification and prediction of oyster freshness.
Fig. 3 shows the overall diagram of the construction methodology of the prediction model for oyster freshness. Response data to oysters at different freshness states are detected by E-nose setup, people’s sensory analyses and physicochemical analyses simultaneously. Features are extracted out from the responses of gas sensor array from the E-nose system, and then optimized by the physicochemical and sensory analyses. A CNN-LSTM model is constructed to predict the freshness or storage time of oysters, and the correlation between the physicochemical indicators and the optimal features is then adopted as the prior knowledge weights to adjust the proposed CNN-LSTM model. Therefore, a PriorBoost CNN-LSTM model is constructed, as shown in the next section.
Fig. 3

    Download: Download high-res image (325KB)
    Download: Download full-size image

Fig. 3. Diagram of the construction methodology of the prediction model for oyster freshness.
2.10. Principle of the priorboost-CNN-LSTM model
The designed PriorBoost-CNN-LSTM model combines data-driven learning with prior knowledge upon an existing CNN-LSTM architecture and incorporating various changes observed during oyster spoilage. Fig. 4 shows the basic workflow of the PriorBoost-CNN-LSTM Model. The model selects the optimal features of each response from 10 sensors as input based on the accuracy reference. Considering the freshness states of the oysters, prior knowledge including putrescine, cadaverine, TVB-N, and sensor-related information are incorporated into the input channel weights of the first convolutional layer to initialize the weights. This helps the model better focus on the key features related to oyster spoilage.
Fig. 4

    Download: Download high-res image (392KB)
    Download: Download full-size image

Fig. 4. The PriorBoost-CNN-LSTM Model.
2.10.1. The CNN-LSTM prediction model
The input is processed through the first convolutional layer, which contains 10 channels and is followed by a batch normalization (BatchNorm) layer to standardize the output. ReLU activation is used to accelerate training and prevent vanishing gradients. The weights of the input channels are dynamically adjusted with the help of the Squeeze-and-Excitation Block mechanism, which enhances the model feature learning capability and optimizes the channel weights by incorporating prior knowledge. The next three convolutional layers follow a similar structure to the first layer, using ReLU activation, max-pooling (MaxPool), and Squeeze-and-Excitation blocks. And the channel number increases progressively as following, the second layer has 60 input channels and 120 output channels; the third layer has 120 input channels and 240 output channels; the fourth layer has 240 input channels and 480 output channels. The convolution operation in each layer dynamically updates the channel weights based on the results of the previous layer. After the fourth convolutional layer, a Dropout layer with a rate of 0.5 is added to prevent overfitting. In each layer of the channel, the prior weights are combined with the weights learned through global information to reconstruct new channel weights that propagate downwards.
The complete data series of each measurement, with a size of 10 × 600, undergoes feature extraction processing before being input into the CNN layers, followed by convolution operations to extract local features. From an overall perspective, each measurement is a sampling test of the E-nose responding to the odors released by the oysters at different freshness states, and thus, the information contained is related to the freshness of the oysters. Specifically, the freshness of the oysters changes over time, and the current freshness state is closely related to its previous states. From a local perspective, each test is essentially a time series, containing the gas sensor responses and recovery process to detect the oyster freshness. The response recovery signals of the gas sensors are key features related to the target odors. Therefore, accurately modeling these temporal dependencies is crucial for improving the classification and prediction accuracy of oyster freshness.
Hence, the extracted features from the CNN layers are then passed to the LSTM layer. The LSTM has been proved particularly adept at handling and retaining long-term sequence dependencies (Panchbhai et al., 2025a). Through its unique memory cells, which consist of gates such as the forget gate, input gate, and output gate, the LSTM can effectively maintain and update historical information within the sequence data, thereby capturing the long-term dependencies in the evolution of oyster freshness. The forget gate discards irrelevant historical information, the input gate determines which new information should be incorporated into the current memory, and the output gate utilizes the historical data to make predictions about the current state. By processing the features extracted by the CNN, the LSTM layer can identify long-term trends, cyclical patterns, and other temporal dynamics in the freshness evolution of oysters, all of which are critical for accurate classification. The integration of the LSTM enables the model to understand and retain historical patterns in the oyster freshness data, significantly enhancing the accuracy of classification predictions for current inputs. As a result, the inclusion of the LSTM layer is essential, as it allows for the effective utilization of CNN-extracted features in the classification task, ensuring more precise predictions of oyster freshness.
2.10.2. The prior boost mechanism
Traditionally, the above CNN-LSTM recognition model will be trained only by the acquired E-nose response data, with some of the prior knowledge as the freshness determinant or target labels, but the accuracy is difficult to improve. In order to fully using the prior information to construct a high precision prediction model, the prior information is adopted to into the training procedure to adjust the weights of the network more meaningfully, which is a direct boost.
The boost by prior weights is performed with the Squeeze-and-Excitation Block mechanism. First, the Kendall coefficients between BAs information, TVB-N, and each sensor response signal are used as the primary component of the prior weights. In this study, cadaverine and putrescine were also defined as one of the classification criteria between the Fresh and Mild Spoilage states, thus the Kendall coefficients of cadaverine and putrescine were selected for calculation. Weight values to each corresponding sensor information was assigned based on the Kendall coefficient between TVB-N and BAs, and then normalized. The first convolutional layer had ten input channels, corresponding to the ten sensors in the E-nose array. The weight 𝜔⁢(𝜊)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟
for channel 𝑜
is calculated according to Eq.(1), where O is the total number of channels, 𝑐⁡(𝜊)
is for cadaverine, 𝑝⁡(𝜊)
is for putrescine, and 𝑇⁡(𝜊)
is for TVB-N.(1)
𝜔⁢(𝜊)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟=
𝑐⁡(𝜊)⋅𝑝⁡(𝜊)+𝑇⁡(𝜊)
∑𝑂
𝑜=1[𝑐⁡(𝜊)⋅𝑝⁡(𝜊)+𝑇⁡(𝜊)]
Subsequently, the Squeeze-and-Excitation mechanism was modified and integrated. This mechanism primarily assigns varying levels of importance to each feature channel through the processes of channel compression (Squeeze) and channel excitation (Excitation). The channel weights produced by the Squeeze-and-Excitation mechanism were combined with the prior weights and applied to the input channels. The final weighted feature value for channel o, denoted as
̃
𝑋𝑜
, was calculated by Eq.(2) as follows,(2)
̃
𝑋𝑜
=
̃
𝜔
𝑜⋅𝑋𝑜=[𝜔⁢(𝑜)𝑠⁢𝑒⋅𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟]⋅𝑋𝑜
where𝑋𝑜
is the previous output feature of channel o,
̃
𝑋𝑜
is the reweighted output feature，
̃
𝜔𝑜
is the attention weight,𝜔⁢(𝑜)𝑠⁢𝑒
is the channel attention weight calculated through the Excitation stage, and𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟
is the weight obtained through prior information.
In subsequent processing, the entropy regularization was and defined as a new loss function. Suppose Lbase represent the base loss of the model, which is the cross-entropy loss to measure the error between the predicted output of the model and the true labels, calculated as Eq.(3), where𝑁
represents the number of samples, 𝐶
represents the number of classes,𝑦𝑖,𝑐
is the true label of sample𝑖,∈{1,2,…𝑁}
corresponding to class𝑐,∈{1,2,…𝐶}
, and𝑝𝑖,𝑐
is the probability of sample 𝑖
being predicted as class𝑐
by the model.(3)
𝐿𝑏⁢𝑎⁢𝑠⁢𝑒=−
1
𝑁
⁢
𝑁
∑𝑖=1
𝐶
∑𝑐=1
𝑦𝑖,𝑐⁢log⁡(𝑝𝑖,𝑐)
Suppose𝐻
represent the entropy value of the prior weight 𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟
, calculated as in Eq.(4)(4)
𝐻=−
𝑂
∑𝑜=1
𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟⁢log⁡[𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟+𝜀]
where 𝜔⁢(𝑜)𝑃⁢𝑟⁢𝑖⁢𝑜⁢𝑟
represents the final prior weight of the channel o.𝜀
is a very small positive number used to prevent situations with log⁡(0)
stops. The higher the entropy value of 𝐻
is, the more uniform the distribution of weights will be. The lower the entropy value is, the more concentrated the distribution of weights will be.
Entropy regularization plays a key role in the loss function by constraining the weight distribution, preventing the model from overly relying on certain feature channels, thereby enhancing stability and generalization ability. It encourages the weights of feature channels to remain balanced, avoiding excessive weights on specific features, which would increase the risk of overfitting. In order to constrain the prior weights during training, preventing the model from overly focusing on specific features while ignoring others, the entropy regularization is performed as shown in Eq.(5).(5)
𝐿=𝐿𝑏⁢𝑎⁢𝑠⁢𝑒+𝜆⋅𝐻
𝜆
controls the weight of the entropy regularization, determining the extent of its influence on the overall loss. In this study, 𝜆
was set to 0.1.
Based on the above mechanism, the model integrates both the channel importance learned from the data and prior knowledge to determine the final weighted coefficients. This allows the model to not only rely on the data to evaluate feature importance but also incorporate external knowledge, improving task performance. Through prior weights, the model can adaptively adjust the channel weights and effectively focus on the most relevant features, improving performance across different data samples and enhancing generalization ability.
3. Results and discussion
3.1. Results of sensory evaluations
The sensory evaluation results are shown in Table 2. It can be seen that the sensory attributes of oysters showed significant changes with prolonged storage. For G1 stored at 4°C, spoilage occurred slowly, with oysters remaining normal for 1–2 days, gradually turning yellow after 6 days, and showing an increase in odor. By days 9–10, they reached the fully spoiled stage. Lower storage temperatures can slow down the spoilage process, which is consistent with previous studies (Min et al., 2020). About G2 stored at 12°C, color began to change to light yellow after 2 days, with a stronger odor by day 6, indicating noticeable spoilage. About G3 stored at 20°C, oysters turned yellow within 24 h, developed a strong foul odor after 48 h, and were fully spoiled by 60 h. About G4 stored at 28°C, oysters began to turn yellow within 12 h, and by 36 h, a strong fishy odor emerged. Higher temperatures accelerated the spoilage process, as anticipated. The increase in microbial activity and enzymatic reactions at elevated temperatures was reflected in the changes in color and odor. These results are consistent with the findings of numerous studies in studying the shelf life of oysters and other seafood (Costa et al., 2014, La Peyre et al., 2018). The color assessment has also been confirmed as an effective method for detecting microbial contamination and adulteration in food (Keskin et al., 2022).

Table 2. Sensory evaluation results of 4 groups of oysters.
Recording time
(G1(Day)/G2(Day)/
G3(Hour)/G4(Hour))	G1 at 4℃	G2 at 12℃	G3 at 20℃	G4 at 28℃
1 / 1 / 1 / 1	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed
2 / 2 / 12 / 4	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed	White with a slight yellow tint;
Normal odor, no unusual smell;
Does not fully recover after pressing	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed
3 / 3 / 24 / 8	Milky white color;
Normal odor, stronger fishy smell;
Quickly rebounds when pressed	White with a slight yellow tint;
Normal odor, stronger fishy smell;
Does not fully recover after pressing	White with a slight yellow tint;
Normal odor, no unusual smell;
Does not fully recover after pressing	Milky white color;
Normal odor, no unusual smell;
Quickly rebounds when pressed
4 / 4 / 36 / 12	Milky white color;
Normal odor, stronger fishy smell;
Quickly rebounds after pressing	Distinct yellow color;
Normal odor, stronger fishy smell;
Does not fully recover after pressing	White with a slight yellow tint;
Normal odor, no unusual smell;
Does not fully recover after pressing	White with a slight yellow tint;
Normal odor, no unusual smell;
Does not fully recover after pressing
5 / 5 / 48 / 16	Milky white color;
Normal odor, stronger fishy smell;
Quickly rebounds after pressing	Distinct yellow color;
Slightly fishy odor;
Does not fully recover after pressing	White with a slight yellow tint;
Slightly fishy odor;
Does not fully recover after pressing	White with a slight yellow tint;
Normal odor, no unusual smell;
Does not fully recover after pressing
6 / 6 / 60 / 20	White with a slight yellow tint;
Slightly fishy odor;
Does not fully recover after pressing	Distinct yellow color;
Strong fishy odor;
Does not recover after pressing	Distinct yellow color;
Strong fishy smell;
Does not recover after pressing	White with a slight yellow tint;
Slightly fishy odor;
Does not fully recover after pressing
7 / 7 / 72 / 24	White with a slight yellow tint;
Slight fishy smell;
Does not fully recover after pressing	Distinct yellow color;
Strong fishy odor;
Poor elasticity, lacks shape	Distinct yellow color;
Strong fishy odor;
Poor elasticity, lacks shape	White with a slight yellow tint;
Slightly fishy odor;
Does not fully recover after pressing
8 / - / - / 28	Distinct yellow color;
Slight fishy odor;
Does not fully recover after pressing			Distinct yellow color;
Slight fishy odor;
Does not fully recover after pressing
9 / - / - / 32	Distinct yellow color;
Strong fishy odor;
Poor elasticity, lacks shape			Distinct yellow color;
Slight fishy odor;
Does not fully recover after pressing
10 / - / - / 36	Distinct yellow color;
Strong fishy odor;
Poor elasticity, lacks shape			Distinct yellow color;
Strong fishy smell;
Does not recover after pressing
- / - / - / 40				Distinct yellow color;
Strong fishy smell;
Does not recover after pressing
- / - / - / 44,48				Distinct yellow color;
Strong fishy odor;
Poor elasticity, lacks shape
3.2. Results of the analysis of TVC, TVB-N, and BAs
According to the National Standard "GB 10136–2015" (National Health and Family Planning Commission of the People's Republic of China, 2015) which specifies the TVC and TVB-N content standards for animal-based aquatic products, as well as several other relevant food safety and quality regulations, the threshold for TVC is set at 5 × 105 CFU/g, and the threshold for TVB-N is set at 30 mg/100 g. TVC and TVB-N are also commonly used as essential indicators for oyster freshness detection (Liu et al., 2024, Su et al., 2024). The analysis results are shown in Fig. 5, with different colors representing different experimental temperatures, and the red dashed line indicating the corresponding thresholds.
Fig. 5

    Download: Download high-res image (671KB)
    Download: Download full-size image

Fig. 5. Variations of (a) TVC (b) TVB-N on storage time at 4℃, 12℃, 20℃, 28℃.
It can be seen from Fig. 5 that, for Group G1 at 4°C, the TVC does not exceed the set threshold in the previous 5 days but shows a sharp increase at Day 7, surpassing 105 CFU/g. The TVB-N content remains below the threshold for up to 8 days, but exceeds the threshold at Day 9. For Group G2 at 12°C, the TVC exceeds the threshold after 3 days, and increases rapidly with time. The TVB-N content approaches the threshold at Day 5 and exceeds it in the following days. For Group G3 at 20°C, the TVC is close to the threshold at Hour 24 and exceeds it after36 hours. The TVB-N content increases rapidly after 48 h and exceeds the set threshold at Hour 60. For Group G4 at 28°C, TVC approaches the threshold within 24 h and exceeds the threshold after 28 h. After Hour 40, TVB-N content increases rapidly, surpassing the set threshold after 44 h.
BAs are toxic to humans and their concentrations are higher in spoiled oysters, making them a common indicator for freshness detection (An et al., 2015, Ruiz-Capillas and Herrero, 2019). The growth of several common BAs were obtained by above experiments, as shown in Fig. 6, which illustrates the variations of total biogenic amines over time at different temperatures. It can be seen that BAs concentrations increase as oysters spoils, with higher temperatures accelerating their accumulation. At 4°C, spermine and spermidine remains stable, while cadaverine and putrescine levels increase, with putrescine rising sharply after 5 days. Histamine was only detected on Day 10. At 12°C, spermine and spermidine stay stable, while histamine, cadaverine, and putrescine increase with a faster rise after Day 3. At 20°C, spermine remains stable, while spermidine increases slightly later, and histamine, cadaverine, and putrescine risese rapidly after 36 h. At 28°C, the levels of spermidine and spermine remain relatively stable, while histamine and putrescine concentrations increase rapidly after 24 h. After 40 h, tyramine begins to appear and its concentration continues to rise over time. In summary, cadaverine and putrescine exhibit similar growth patterns across all temperatures, while other amines show temperature-dependent variations.
Fig. 6

    Download: Download high-res image (451KB)
    Download: Download full-size image

Fig. 6. Variations of BAs content at (a) 4°C (b) 12°C (c) 20°C (d) 28°C (ppm, parts per million, 1 ppm corresponds to 1 μg/g).
3.3. Data processing and freshness prediction results
3.3.1. Freshness level definition and data distribution
Based on the National Standard of the People's Republic of China, "GB 10136–2015," (National Health and Family Planning Commission of the People's Republic of China, 2015), along with the experimental results of BAs in this study and sensory evaluations, the spoilage process of oysters is classified into four rough and distinct freshness levels. These levels correspond to the data collected by the E-nose over time. The classification standards can be found in Table 3. In this context, the BAs in Table 3 refer to the contents of cadaverine and putrescine.

Table 3. Definition of rough freshness levels of oysters.
Freshness
level	Definition	TVB-N (mg/100 g)	TVC
(CFU/g)	BAs
(ppm)	Sensory
evaluation
Fresh	Very fresh, with all indicators within the acceptable range	＜30	＜5 × 10⁵	＜10	Fresh
Mildly Spoiled	Slight sensory changes, with all indicators within the acceptable range	＜30	＜5 × 10⁵	＜10	Fresh or slight color changes
Clearly Spoiled	Noticeable spoilage in sensory attributes, with some indicators exceeding the acceptable range	≥ 30 or 5 × 10⁵	≥ 10	Noticeable signs of spoilage
Severely Spoiled	Very obvious spoilage in sensory attributes, with all indicators exceeding the acceptable range	≥ 30	≥ 5 × 10⁵	≥ 10	Very evident signs of spoilage
TVB-N (total volatile basic nitrogen); TVC (total viable count); BAs (biogenic amines).
Continuous E-nose odor data collection were conducted at four experimental temperatures, obtaining a total of 1239,000 valid data points. Each measurement, corresponding to a measuring cycle above, consists of 600 data points, resulting in 1860 valid measuring cycles. Among them, there are 875 measurements at 4°C, 690 measurements at 12°C, 295 measurements at 20°C, and 205 measurements at 28°C. Table 4 shows the distribution of measurements across the four freshness levels at different temperatures.

Table 4. Number of measuring cycle data at four freshness levels and four temperatures.
Freshness level	G1 at 4℃	G2 at 12℃	G3 at 20℃	G4 at 28℃
Number of cycles	Storage
time	Number of cycles	Storage
time	Number of cycles	Storage
time	Number of cycles	Storage time
Fresh	170	Day 1–2	120	Day 1	100	Hour 0–24	52(0−12)	Hour 0–12
Mildly Spoiled	240	Day 3–5	120	Day 2	50	Hour 25–36	53(13−24)	Hour 13–24
Clearly Spoiled	279	Day 6–8	235	Day 3–5	100	Hour 37–60	68(25−40)	Hour 25–40
Severely Spoiled	186	Day 9–10	215	Day 6–7	45	Hour 61–72	32(41−48)	Hour 41–48
Total samples	875	690	295	205
Definitions of Fresh, Mildly Spoiled, Clearly Spoiled, and Severely Spoiled refer to Table 3.
3.3.2. E-nose feature extraction and evaluation
Feature extraction was performed on the initial samples to reduce processing complexity and improve classification efficiency, a commonly used method in E-nose data processing (Yan et al., 2015, Yang et al., 2023). Five features were selected: Wavelet energy (WE), Response area (RA), Steady-state mean response (SR), Euclidean distance (ED), Response peak (RP). The WE value refers to the sum of the squares of the wavelet coefficients at each level, which is a measure of the sensor signal frequency components across different scales using wavelet transform. For a sensor j, if a three-level Daubechies wavelet decomposition is applied to the signal of the sensor. Then the WE feature of sensor j is obtained as𝑊⁢𝐸𝑗=∑3
𝐽=1∑𝑛
𝑖=1|𝑧𝑗⁢𝐽⁡(𝑖)|2
, j representing the total wavelet energy value of sensor j, where J represents the level of wavelet decomposition, here the total number of levels is 3, n represents the number of coefficients at each level, and zjJ (i) represents the corresponding wavelet decomposition coefficients. The RA is obtained by integrating each sensor response curve with the baseline every 600 s using the trapezoidal method. Suppose RAj represent the RA of the j-th sensor, and Sj(ti) represent the response value of the sensor j at time ti. Its computation is by 𝑅⁢𝐴𝑗=∫𝑡600
𝑡1𝑆𝑗⁡(𝑡𝑖)𝑑𝑡
, as the response time ti is from t1 to t600. Steady state response value is the response value approaching to the steady state of signals over time. Suppose SRj represent the steady-state response value of the j-th sensor, and xij represent the value of the j-th sensor at the i-th data point. The calculation is performed over a time interval of length n, which corresponds to the range from the 150th to the 350th data points, as 𝑆⁢𝑅𝑗=
1
𝑛
⁢∑350
𝑖=250𝑥𝑖⁢𝑗
, Here n equals to 101. The Euclidean distance is calculated using the coefficients from the three-level Daubechies 4 wavelet decomposition of the sensor response signal based on the first complete response. To avoid negative values, the sum of squares is used. Suppose EDj represent the Euclidean distance of the sensor j, J represent the number of decomposition levels, bJ,j represent the calculated wavelet coefficient at level J, and aJ,j represent the wavelet coefficient of the reference signal, which is the first complete sensor response at the start of the odor test. It is computed by 𝐸⁢𝐷𝑗=√∑3
𝐽=1(|𝑏𝐽,𝑗−𝑎𝐽,𝑗|2)
. The peak response (PR) refers to the maximum value within the 600 s response curve of the sensor.
These features were then used for classification based on the freshness classification standard defined 3.3.1. And the above 5 kinds of features of E-nose data were calculated accordingly and input to the CNN-LSTM model. The model performance was evaluated using ten-fold cross-validation, accurately distinguishing the four freshness levels of the oyster samples. The final results, shown in Table 5, indicate that the data using WE value features achieved the best classification performance.

Table 5. Accuracy (%) of different features at four freshness levels.
Feature	WE	RA	SR	ED	RP
Fresh	98.14	97.97	98.11	93.95	96.05
Mildly Spoiled	97.87	95.36	96.52	86.27	97.10
Clearly Spoiled	97.77	98.07	95.53	87.97	95.70
Severely Spoiled	98.69	97.19	95.71	94.22	98.32
Overall Precision	98.11	97.19	96.37	90.41	96.80
WE (Wavelet energy); RA (Response area); SR (Steady-state mean response); ED (Euclidean distance); RP (Response peak). Definitions of Fresh, Mildly Spoiled, Clearly Spoiled, and Severely Spoiled refer to Table 3.
The WE value achieved the highest overall accuracy at 98.11 %, while ED performed the worst at 90.41 %. For the four freshness levels, WE achieved the highest classification accuracy of 98.14 %, 97.87 %, and 98.69 % for fresh, mildly spoiled, and severely spoiled categories, respectively. The RA achieved the highest accuracy of 98.07 % for the obvious spoilage category.
3.4. Prior knowledge analysis
The WE feature of each response curve in the array was extracted out to represent the sensor response and used as the following input of the pattern recognition models. In order to further understand the relationship between the sensors and the freshness states, the prior knowledge of the TVB-N, BAs and TVC results were further studied. The Kendall coefficient was utilized to evaluate relationship between the sensor response and the results of TVB-N and BAs, as shown in Fig. 7.
Fig. 7

    Download: Download high-res image (435KB)
    Download: Download full-size image

Fig. 7. Correlation between sensors and BAs, TVB-N at (a) 4°C (b) 12°C (c) 20°C (d) 28°C.
In Fig. 7, red indicates a positive correlation, while blue denotes a negative correlation. The size of the hexagons is proportional to the correlation strength, with larger hexagons representing stronger correlations and smaller hexagons indicating weaker correlations. It can be seen that at 4°C, TVB-N, cadaverine, and putrescine exhibit strong positive correlations with sensors S2, S9, and S10. At 12°C, the correlation between biogenic amines and sensor responses becomes more pronounced, with sensors S1, S2, S3, S4, and S8 showing strong positive correlations with the trends of cadaverine, putrescine, and histamine. At 20°C, sensors S1, S2, and S6 display strong correlations with TVB-N, cadaverine, putrescine, histamine, and tyramine. At 28°C, the correlation between TVB-N and the wavelet energy values of the sensors becomes more evident. Sensors S3, S4, S7, S8, S9, and S10 exhibit strong positive correlations with the trends of cadaverine and histamine. However, the correlation between putrescine and the wavelet energy features of the sensors at 28°C is slightly weaker compared to cadaverine and histamine.
Correlation analysis revealed a significant relationship between the physicochemical freshness indicators and the responses of the E-nose sensors, as well as with temperature and spoilage duration. Fig. 7 has shown the strong correlation of TVB-N, putrescine, and cadaverine as indicators of spoilage in fresh food with the sensors, hence, this prior information is valuable to be further investigated in assessing food freshness. This similar relevance between the physicochemical indicators has been established in evaluating the freshness of fish and meat products (Costa et al., 2018, Veciana-Nogués et al., 1997). Therefore, both cadaverine and putrescine were consistently detected across all experimental temperatures, with their accumulation accelerating at specific time points. And the TVB-N, cadaverine, and putrescine are chosen as key indicators for assessing the freshness of oysters at more refined time intervals. The Kendall coefficients between the variations in the concentrations of TVB-N, cadaverine, and putrescine during the experiment and the WE feature variations of the ten sensors were computed. Sensors that demonstrated a strong correlation with TVB-N, cadaverine, and putrescine, were assigned higher initial weights and given greater attention. The variation patterns of TVB-N, cadaverine, and putrescine throughout the experiment were mapped onto these highly correlated sensors, thereby enhancing their contribution to the subsequent classification process.
3.5. Evaluation of the PriorBoost-CNN-LSTM model
A ten-round five-fold cross-validation was conducted on the acquired data from four temperature conditions, with each fold training for 100 epochs. K-fold cross-validation is an effective method widely used for model performance evaluation, capable of effectively testing model performance in various scenarios (Panchbhai et al., 2025b). To compare the effect of prior information boosting, the classical CNN-LSTM (CL) were tested under same environment with the verification environment of the proposed PriorBoost-CNN-LSTM (PBCL). All the results of precision, recall, F1 score, and accuracy across different time intervals and temperatures have been calculated and shown in Table 6, Table 7. The formulas for precision, recall, F1 score, and accuracy are shown as (6), (7), (8), (9). 𝑇⁢𝑃
represents true positive class and model predicted positive class,𝐹⁡𝑃
represents the true negative class but the model predicts a positive class, 𝐹⁡𝑁
represents a true positive class but the model incorrectly predicts a negative class, 𝑇⁢𝑁
represents the true negative class and the model predicts a negative class.(6)
𝑃⁢𝑟⁢𝑒⁢𝑐⁢𝑖⁢𝑠⁢𝑖⁢𝑜⁢𝑛=
𝑇⁢𝑃
𝑇⁢𝑃+𝐹⁡𝑃
(7)
𝑅⁢𝑒⁢𝑐⁢𝑎⁢𝑙⁢𝑙=
𝑇⁢𝑃
𝑇⁢𝑃+𝐹⁡𝑃
(8)
𝐹⁡1⁢𝑆⁢𝑐⁢𝑜⁢𝑟⁢𝑒=2⋅
Precision⋅Recall
Precision+Recall
(9)
𝐴⁢𝑐⁢𝑐⁢𝑢⁢𝑟⁢𝑎⁢𝑐⁢𝑦=
𝑇⁢𝑃+𝑇⁢𝑁
𝑇⁢𝑃+𝑇⁢𝑁+𝐹⁡𝑃+𝐹⁡𝑁

Table 6. Classification results with 1- and 2-hour intervals.
Prediction resolution	Storage temperature	Models	Precision (%)	Recall (%)	F1 Score (%)	Accuracy (%)
1-hour	4℃	CL	63.45 ± 0.74	62.42 ± 0.97	60.99 ± 1.26	62.42 ± 0.97
PBCL	72.79 ± 1.10	71.44 ± 1.27	70.61 ± 1.08	71.44 ± 1.27
12℃	CL	72.64 ± 2.09	71.17 ± 1.76	70.42 ± 1.93	71.17 ± 1.76
PBCL	83.48 ± 3.11	82.54 ± 1.96	82.09 ± 2.31	82.54 ± 1.96
20℃	CL	72.62 ± 1.83	71.87 ± 1.18	71.11 ± 1.26	71.87 ± 1.18
PBCL	85.54 ± 2.44	84.00 ± 1.51	83.65 ± 2.06	84.00 ± 1.51
28℃	CL
PBCL	75.51 ± 1.80
88.44 ± 2.99	72.37 ± 1.10
86.84 ± 2.95	73.40 ± 1.41
86.49 ± 2.95	73.33 ± 1.21
86.84 ± 2.95
2-hour	4℃	CL	83.06 ± 0.84	81.83 ± 0.98	81.57 ± 0.98	81.57 ± 0.98
PBCL	86.95 ± 1.03	86.34 ± 1.15	86.05 ± 1.09	86.34 ± 1.15
12℃	CL	81.26 ± 0.98	80.25 ± 0.88	80.04 ± 0.97	80.25 ± 0.88
PBCL	90.07 ± 2.38	89.83 ± 2.29	89.92 ± 2.00	89.83 ± 2.29
20℃	CL	82.23 ± 1.06	81.25 ± 1.25	81.16 ± 1.19	81.25 ± 1.25
PBCL	90.19 ± 2.01	89.24 ± 1.81	89.44 ± 1.85	89.24 ± 1.81
28℃	CL	83.88 ± 2.88	81.60 ± 2.31	80.75 ± 2.19	81.60 ± 2.31
PBCL	90.72 ± 1.86	90.00 ± 1.79	89.94 ± 1.80	90.00 ± 1.79
CL(CNN-LSTM); PBCL(PriorBoost-CNN-LSTM)

Table 7. Comparison with existing studies.
Methods	Main measurement	Freshness precision	Freshness determinant	Storage temperature	Reference
GC-MS combined with multivariate statistical analysis	Volatile Organic Compounds	1 day	changes in volatile compounds	5℃ /15℃	Park et al. (2024)
Analysis of biochemical properties	Microbial analysis, adenosine triphosphate	1 day	biochemical properties	−1℃	Dong et al. (2023)
Deep learning method fused with MDA and SH information	Oyster meat images	1 day	freshness category of 4 levels	4℃	Lu et al. (2023)
Determining freshness based on WeChat Mini Program	Color change of curcumin film	1 day	color changes in curcumin film	4℃	Chen et al. (2023)
Modeling freshness prediction using Partial Least Squares	Colorimetric sensor array image data	1 day	TVC content	4℃	Wang et al. (2024)
A Prior Knowledge Boosted CNN-LSTM Prediction Model	E-nose data	1 h / 2 h	Direct determination of storage duration	4℃/12℃ /20℃/28℃	This work
As presented in Table 6, the PBCL model with the prior boosting mechanism is superior to the unboosted model. At the four experimental temperatures, the most significant improvement in accuracy was observed when taking one-hour interval freshness evaluation. The accuracies of PBCL model at the four experimental temperatures were 71.44 %, 82.54 %, 84.00 %, and 86.84 %, representing increases of 9.02 %, 11.37 %, 12.13 %, and 13.51 %, respectively, compared to the classical CL results. Notably, as the experimental temperature increased and the number of sample categories decreased, the improvement effect became more pronounced. When using two-hour intervals as the classification resolution, the accuracies of PBCL at the four experimental temperatures increased from 81.57 %, 80.25 %, 81.25 %, and 81.60–86.34 %, 89.83 %, 89.24 %, and 90.00 %, representing improvements of 4.77 %, 9.58 %, 7.99 %, and 8.40 %, respectively.
Fig. 8 illustrates the five-fold cross-validation accuracy of the PriorBoost-CNN-LSTM model across different temperature oyster odor datasets, with the results falling within the model's performance range. In Fig. 8, experiments of the freshness evaluation intervals of 1-hour and 2-hour at 4°C, 12°C, 20°C, and 28°C were performed shown as (a), (b), (c) and (d) respectively. These results indicate that the PriorBoost-CNN-LSTM significantly enhanced the classification accuracy of E-nose samples at various temperatures, particularly in scenarios involving more refined time intervals. However, it can also be observed that the accuracy for the 1-hour interval samples at 4°C storage conditions is relatively lower. This is due to the slower odor changes in oysters stored at low temperatures. As indicated by the previous sensory evaluations and various analytical results, oysters stored at 4°C have a longer shelf life, and their odor changes are less pronounced over short periods. This leads to the lower accuracy under these conditions. When the time interval is extended to two hours, the classification accuracy shows a significant improvement.
Fig. 8

    Download: Download high-res image (291KB)
    Download: Download full-size image

Fig. 8. Five-fold cross-validation accuracy of the PBCL model for classification of samples with 1-hour (1 h) and 2-hour (2 h) intervals, showing performance within an acceptable range. (a) 4°C (b) 12°C (c) 20°C (d) 28°C.
The confusion matrix diagrams from five-fold cross-validation was generated to analyze the classification changes at specific time points for each temperature, with hourly intervals as classification categories. In the complete confusion matrix, the predictions generally show no significant temporal deviation from the true classes, with misclassified samples primarily concentrated around adjacent time points of the actual classes. Given the large number of categories classified by hour in both fresh and mildly spoiled states, four time periods with high misclassification rates across different temperatures were presented, which are the mid-stage (39–46 h) at 4°C, the late-stage (36–42 h) at 12°C, and the early stages (1–8 h) at both 20°C and 28°C to compare the classification results of the CNN-LSTM model and the PriorBoost-CNN-LSTM model at the same time points, as shown in Fig. 9.
Fig. 9

    Download: Download high-res image (548KB)
    Download: Download full-size image

Fig. 9. (a) CL model classification at 4°C (39–46 h) (b) PBCL model classification at 4°C (39–46 h) (c) CL 12°C (36–42 h) (d) PBCL 12°C (36–42 h) (e) CL 20°C (1–8 h) (f) PBCL 20°C (1–8 h) (g) CL 28°C (1–7 h) (h) PBCL 28°C (1–7 h).
Fig. 9(a) shows the classification results of the CNN-LSTM model under 4°C conditions from the Hour 39–46. It can be observed that during this period, the model's predictions are mostly concentrated around adjacent time points. In contrast, Fig. 9(b) shows the classification results of the PriorBoost-CNN-LSTM model, which demonstrates a significant improvement, particularly from the Hour 40–44, with predictions being more focused. A similar trend can be observed in Fig. 9(c) and (d). Under 12°C conditions, from the Hour 36–41, the PriorBoost-CNN-LSTM model clearly outperforms the original model in classification accuracy, indicating stronger classification capability. In the early stage of the 20°C experiment, the classification results of both models are shown in Fig. 9(e) and (f), respectively. Similarly, in the early stage of the 28°C experiment, the classification results of the two models are presented in Fig. 9(g) and (h). It is evident that the PriorBoost-CNN-LSTM model outperforms the original CNN-LSTM model, significantly reducing the occurrence of misclassifications.
3.6. Analysis and discussion of the PriorBoost-CNN-LSTM Model results
The above results proved the effectiveness of the proposed PriorBoost-CNN-LSTM model based on a self-developed E-nose system to predict the freshness of oysters. Real-time hourly-level freshness prediction is achieved through the E-nose system, demonstrating promising potential for online oyster freshness detection. The feasibility of the proposed classification and prediction method is validated under varying temperature conditions, providing new insights for the future application of E-nose technology in seafood freshness detection.
Typical methods in existing studies are summarized and compared with the proposed method in Table 6, including the GC-MS combined with multivariate statistical analysis for detecting VOC changes (Park et al., 2024), the analysis of biochemical properties through microbial analysis and adenosine triphosphate (Dong et al., 2023), deep learning methods fused with MDA and SH information from oyster meat images (Lu et al., 2023), determining freshness based on a WeChat program using the color change of curcumin film (Chen et al., 2023), and modeling freshness prediction using partial least squares with colorimetric sensor array image data to predict TVC content (Wang, B et al., 2024).
Compared to the GC-MS and microbial analysis methods, the proposed E-nose method for detecting oyster freshness offers several advantages. First, it requires less stringent environmental conditions, and it does not rely on expensive equipment or sterile environments, and it has relatively simple operational requirements that do not necessitate specialized, rigorous training. In addition, compared to image recognition-based methods, the collection of odor data is simpler than capturing standard images. The E-nose does not require specific angles or lighting conditions for data acquisition, and its compact and lightweight sensor board is portable and easy to deploy.
And it can be seen that most existing methods offer freshness precision at a daily level, while this study achieves hourly-level precision, allowing for a more accurate representation of changes in oyster freshness. This improvement is particularly significant for detecting the freshness of oysters, which have a short shelf life, especially when stored at higher temperatures such as 12°C, 20°C, and 28°C. Within a 2-hour interval, the classification accuracy for oysters at 12°C, 20°C, and 28°C in this study reaches approximately 90 %. Thus, the freshness detection precision is significantly enhanced in this work.
Additionally, compared to commercially available E-noses, our self-developed E-nose offers several significant advantages, including lower costs, greater flexibility in sensor model replacement, and the ability to adjust sampling frequency and power supply voltage. These features enhance its adaptability to various real-time detection requirements. Moreover, the system allows for customization of sensor configurations, making it particularly well-suited for specific application scenarios, such as oyster freshness monitoring in logistics.
To enhance the practical applicability of the E-nose system, it has been developed to be more portable, with a higher degree of integration and real-time capabilities, designed to meet the deployment requirements of edge devices and accommodate various detection scenarios (Wei et al., 2025). Accordingly, with the proposed model, oyster freshness is evaluated based on an E-nose system that is low-cost, easily portable, and highly flexible. Therefore, the system can be easily integrated into warehousing and logistics equipment. It allows for rapid and flexible sensor model replacement, adjustment of sampling frequency, and modification of sensor power supply voltage to meet different detection requirements. The total measurements can be implemented automatically, without any human involvement, showing great advancements. And particularly, the freshness statement is not only defined into 4 levels, but also defined more directly as the duration time from the most fresh stage. The freshness evaluation result is not a measurement of any of the physicochemical indicators, but a fusion of E-nose sensor responses and the physicochemical variables. By the validation of experiments at 4 different temperatures, the method proposed is more adaptable to a broader range of temperature conditions and simulated oyster storage and sales scenarios at different temperatures.
Moreover, the integration of non-destructive testing technologies with software tools and data analysis methods is of great importance in specific application fields. Through multi-dimensional data fusion and collaborative analysis of detection models, the practicality of the detection methods can be significantly enhanced (Arslan et al., 2025, Zhu et al., 2024, Zhang et al., 2004). As a significant extension of non-destructive analytical techniques, the core advantage of the electronic nose lies in its ability to specifically detect volatile compounds, and this advantage can be further amplified through integration with commonly used software tools and data analysis methods in the non-destructive analysis field.
4. Conclusion
A PriorBoost-CNN-LSTM model with an E-nose system for oyster freshness evaluation has been proposed and thoroughly studied. The correlation between the oyster spoilage information and the response from our self-developed E-nose array, the changes in sensory quality, TVB-N, TVC, and BAs of oysters under different storage temperatures were systematically detected, and analyze. This trend information of physicochemical indicators was adopted and performed on the weight correction in the classification prediction model PriorBoost-CNN-LSTM and a novel freshness prediction and classification method was proposed combining multi-indicator information with the E-nose system. Systematically experiments have been performed to verify the effectiveness, performance of the E-nose technology and comparisons have been made.
It has been proved that the proposed PriorBoost-CNN-LSTM model shows a new way to build up a high precision food quality measurement model by combining the E-nose data with prior knowledges of TVB-N, BAs, and the sensory records. It integrates multi-dimensional information, and leverages the high correlation between TVB-N, BAs, and sensor array signals to correct and enhance the learning process. It not only significantly improves the classification accuracy, prediction time precision, and reliability, but also avoids the complexities and high costs of traditional detection methods, providing a novel technique for food quality control under varying conditions and spoilage rates. However, the prediction accuracy still needs improvement, as the study examined only four storage states and the adaptability to other types of storage conditions remains a significant challenge. Future research will explore alternative model structures and incorporate more diverse traditional data fusion methods to develop more portable, low-cost E-nose arrays and high-precision, highly adaptable prediction models. Additionally, incorporating more traditional monitoring data, such as color changes, fatty acid variations, and pH measurements, could further enhance the model's ability to predict the freshness of oysters and other seafood. Given the promising results obtained with oysters, this method shows great potential for broader application to other perishable and odor-emitting seafood products. Additionally, the E-nose system is compact and easy to deploy, making it suitable for use in scenarios such as logistics transport vehicles and sales shelves, facilitating real-time freshness detection. Ongoing research will focus on expanding its applicability to other seafood products to find the general rules among them.
CRediT authorship contribution statement
Baichuan Wang: Supervision, Resources, Data curation. Quansheng Dou: Supervision, Methodology. Xiaoshuan Zhang: Supervision, Resources. Guangfen Wei: Writing – review & editing, Project administration, Methodology, Funding acquisition. Xiaolong Lv: Writing – original draft, Software, Investigation, Formal analysis. Jie Zhao: Writing – review & editing, Methodology, Conceptualization. Wei Zhang: Supervision, Data curation.
Authorship confirmation
All authors have agreed to be listed as co-authors and take full responsibility for the content of the manuscript.
Originality statement
This manuscript is the authors’ original work and has not been published previously, nor is it under consideration for publication elsewhere.
All authors have made substantial contributions to the conception, design, execution of the experiments, and/or writing of the manuscript. All authors have read and approved the final version. The authors declare that there is no conflict of interest related to this work.
Funding
This research is supported by the Natural Science Foundation of China (62173213) and the Xinjiang Uygur Autonomous Region Natural Science Foundation project "Research on Key Technologies for Pre-detection of Fruit Quality Based on Micro-environmental Perception Big Data" (Project No. 2022D01A237).
Declaration of Competing Interest
The authors have no conflict of interest to declare.
Data availability
Data will be made available on request.
