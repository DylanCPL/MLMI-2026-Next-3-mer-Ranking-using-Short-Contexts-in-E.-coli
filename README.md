# MLMI-2026-Next-3-mer-Ranking-using-Short-Contexts-in-E.-coli
*Summary of "Next-3-mer Ranking using Short Contexts in *E. coli*", as presented at MLMI 2026, by Dylan Lewis, Madelyn Forrest, Madelin Stueck, and Gurjit Randhawa*

<p align="center">
  <img id="fig:graphical-abstract" src="https://github.com/user-attachments/assets/dc83137b-8461-44bc-bed6-94627708ec2f"alt="Graphical Abstract: summary of 3-mer ranking pipeline" width="80%">
  <br>
  <em><strong>Figure 1:</strong> Graphical abstract summarizing the methodology of our study.</em>
</p>

# Introduction (and the *why*) 
Here is a summary of our work that was presented at MLMI 2026 (held at Rikkyo University in Tokyo). It is titled: Next-3-mer Ranking using Short Contexts in *E. coli*. At the conference, I met some incredible minds, who took the time to genuinely engage with my work and ask some very thoughtful questions (and in return, I got to learn about some very cool ongoing research at campuses across the world). On top of that, we were also incredibly honoured to receive awards for Best Student Paper and Best Presentation! 

So, with all of that in mind, it only seemed appropriate to create this page, to answer the questions that I didn’t fully have the time to answer then. Since the full paper is currently in the process of being published in ACM, here is a recap of the presentation, along with some extra details that I didn’t have time to cover.

First off, the key takeaways:
*	Using only training from a single genome, and only short contexts (<100 letters), models were able to rank likely next 3-mers well above baseline
*	Transformer and LSTM performing better than 1D CNN; all performing better than Markov
*	Top-1 accuracy, and distribution of top-1 predictions poor among all models, though better with neural models, and best with LSTM and transformer.
*	Short DNA contexts contain enough local structure for small neural models to learn how to rank next letters
*	This study is the first benchmark of next *k*-mer ranking by locally-trained models using short contexts
*	Future work will extend this benchmark to larger models and data, enabling the direct comparison between tools for aiding *de novo* genome assembly
*	Such work could have use in de Bruijn graph assembly, at the stage of branch pruning, enabling more computationally efficient pipelines for *de novo* genome assembly when using short reads

Our genome is formed from DNA. DNA is the language of life: it’s the set of instructions that tell your cells what to do, and how. DNA sequences are long strings of molecules with interchangeable pieces, with those pieces being Adenine, Cytosine, Guanine, and Thymine. We can represent DNA sequences digitally as strings over the alphabet A, C, G, T. Genome sequencing is the process that determines the order of those sequences of letters in a genome. When done for the first time without any reference, this is called *de novo* genome sequencing.
That’s great and all, but why does that matter? That is, why is *de novo* genome sequencing important? Well, for one, it’s useful in conservation, where it allows us to catalogue all living species beyond just noting physical traits. As of 2019, less than 1% of all species have had their genomes sequenced [1]. Yet the rate of species lost is accelerating, at 100X what it should be over the last century, likely due to human activity [2].
*De novo* genome sequencing is also important for the discovery of novel pathogens, such as during the 2019 pandemic. By sequencing the genome, we can then use that information for diagnostics, determining public health protocols, and for the development of vaccines [27].
Finally, *de novo* genome sequencing is important in terms of understanding mutations. Even within a single species, there can be enough differences between sequences that *de novo* assembly can catch details that would be impossible to catch otherwise [28].

However, in genome assembly, no machine can read an entire DNA sequence at once, they can only read pieces. So, in order to read a DNA sequence, we first have to break it up into pieces [29]. In short-read sequencing, these pieces are between 150 and 300 letters long. To be fair, long read sequencing exists, and is continually improving, however, it has its own problems: cost, throughput, and accuracy, namely [3, 5]. So, we’ll focus on short-read sequencing.
This assembly problem is like if I shredded a document, and then asked you to tape the pieces back together in the right place. In the case of *de novo* genome sequencing, this is like having to put the pieces together, without knowing what the document was, or what it should look like. In terms of the scale of this problem, putting back together the *E. coli* genome (around 4 million letters) would be like putting together a document shredded into 20,000 pieces (not my ideal way to spend a Sunday afternoon, but I’ve never been a huge fan of puzzles).
Now, as for the work we did (and while on the topic of puzzles): when solving a puzzle, wouldn’t it be helpful to have an idea of what pieces are likely going to be beside each other? Well, that’s exactly what information our research tries to gain: using only short contexts from the *E. coli* genome, can a machine learning model learn enough patterns in the data to rank the most likely next letters?

For this project, we created a dataset from the *Escherichia coli* genome (retrieved from NCBI [23]), where each datapoint consisted of 99 letters in total: the first 96 letters as context, and the 3 final letters used as the target label. Note that this gives us datapoints of under 100 letters. Remember earlier, we mentioned that short reads are typically in the 150-300 letter range, meaning that they would have a minimum of 150 letters? That’s not always true, as sometimes, even when we set a target length of 150 letters in sequencing, some reads may end up shorter than that. Using only 96 letters of context (while an arbitrary choice) allows us to investigate that a machine learning pipeline would also be able to include these smaller reads, rather than not being able to use them at all.
In this preliminary study, the models were restricted to local models: that is, no models were allowed any form of pre-training or transfer learning. This gives us a better understanding of how the models perform based on their architecture alone, as the training process remains the exact same for each model. This also, of course, made things much simpler for rapid testing (and re-testing) for this preliminary study, as it meant all of our models were (relatively) lightweight.
Our goal was to benchmark the ability of basic sequential learners to accurately rank the next most likely 3-mers to follow a given context, using only local information. By the way, when I talk about 3-mers, I mean any 3 letter word over the alphabet A,C,G,T.

The figure that you saw at the beginning of this page ([Figure 1](#fig:graphical-abstract), the graphical abstract of our paper) sums our methodology quite simply. For this preliminary study, we use four models: a Markov model, a 1 dimensional CNN, an LSTM, and a transformer model. We used a five-fold 60/20/20 training/validation/testing split. We tried three different tokenization schemes for the 96 letter context (that is, in the input, what the model sees as individual words). In one scheme, we consider each letter a word; in another, every set of 3 letters is a word. In the last one, every non-overlapping set of 3 letters is a word. 
For each fold, and for each tokenization scheme, the models were trained on the three training folds, and early stopping was determined for the three neural models using the validation fold. The models were then evaluated by how they ranked the most likely next 3-mers for the contexts in the test fold.

# Results

Now, how did everything stack up? Overall, the transformer with the overlapping 3-mer tokens performed best, and so for the rest of the results, we will focus on that tokenization scheme. However, for the other two tokenization schemes, the smaller and simpler LSTM actually outperformed the transformer. Below is a table that captures all of those results (in **bold** are the best results for that tokenization scheme; in ***bold and underlined*** are the best results overall).

| Tokenization scheme      | Model       | Top 1  | Top 3  | Top 5  | Top 10 | Top 20 | Mean true rank | Median true rank |
| ----------------- | ----------- | ------ | ------ | ------ | ------ | ------ | -------------- | ---------------- |
| Non-overlapping   | Transformer | 0.0541 | 0.1372 | 0.2079 | 0.3546 | 0.5720 | 20.81          | **17**               |
| Non-overlapping   | LSTM              | **0.0544**      | **0.1377** |**0.2085** | **0.3549** | **0.5732** | **20.74**  | **17**             |
| Non-overlapping   | CNN               | 0.0428      | 0.1126 | 0.1739 | 0.3066 | 0.5164 | 23.08  | 20             |
| Non-overlapping   | Markov            | 0.0314      | 0.0790 | 0.1254 | 0.2279 | 0.4072 | 27.46  | 26             |
| Overlapping       | Transformer | ***0.0578*** | ***0.1457*** | ***0.2191*** | ***0.3694*** | ***0.5893*** | ***20.09***          | ***16***               |
| Overlapping       | LSTM              | 0.0566      | 0.1428 | 0.2157 | 0.3646 | 0.5835 | 20.32  | ***16***             |
| Overlapping       | CNN               | 0.0416      | 0.1094 | 0.1689 | 0.2984 | 0.5066 | 23.44  | 20             |
| Overlapping       | Markov            | 0.0392      | 0.0877 | 0.1334 | 0.2351 | 0.4136 | 27.18  | 25             |
| Single nucleotide | Transformer | 0.0551 | 0.1396 | 0.2113 | 0.3590 | 0.5779 | 20.55          | 17               |
| Single nucleotide | LSTM              | **0.0563**      | **0.1422** | **0.2147** | **0.3633** | **0.5821** | **20.36**  | ***16***             |
| Single nucleotide | CNN               | 0.0402      | 0.1062 | 0.1645 | 0.2918 | 0.4993 | 23.73  | 21             |
| Single nucleotide | Markov            | 0.0392      | 0.0877 | 0.1334 | 0.2351 | 0.4136 | 27.18  | 25             |
  
In [Figure 2(a)](#fig:top-n-acc), we look at the Top-*N* accuracies for the various values of N, from 1 to 64. We compare the models to a uniform baseline (which guesses labels randomly) and to a test-frequency baseline (which uses information from the test set to predict the N-th most frequent 3-mer)
As we can see, the Transformer and LSTM outperform the 1D CNN consistently. These three neural models all consistently outperform the Markov model. All four models outperform the test-frequency baseline.

We then subtracted the test-frequency baseline from these curves, in order to visualize just how much these models outperformed the best possible baseline, as shown in [Figure 2(b)](#fig:top-n-acc). For the LSTM and transformer, they by far do the best at the top 20 mark, scoring nearly 20% above that baseline.

<p align="center">
  <img id="fig:top-n-acc" src="https://github.com/user-attachments/assets/f88915f2-cd82-4ea4-996c-dee040eb7491"alt="Top-N accuracy curves" width="80%">
  <br>
  <em><strong>Figure 2:</strong> Top-N accuracy curves.</em>
</p>

We also visualized where the four models ranked the true label for each test case, as shown in [Figure 3](#fig:rank_distribution). While the Markov model shows a roughly equal distribution of the true label amongst its predictions, the neural models consistently predict the true label to be in a higher rank. 

<p align="center">
  <img id="fig:rank_distribution" src="https://github.com/user-attachments/assets/8eb22676-63bc-4c9c-a41a-8cfab7a5abae"alt="Rank distribution" width="80%">
  <br>
  <em><strong>Figure 3:</strong> Distribution of true 3-mer rank.</em>
</p>

Next off, wanted to see if our models were actually learning the 'big picture' of the *E. coli* genome. To do this, we used a frequency chaos game representation (fCGR) [25]. It’s a really nice visualization used throughout genomic studies, and it acts as a genomic signature: that is, for each species, their fCGR will be unique [26]. The best way to think of it is as a 2D barcode for a DNA sequence, whereby you would expect that closely related species have more similar looking fCGRs.
We generated this 'barcode' image from the actual E. coli sequence, as you can see. If our models are predicting things accurately, their predictions should be able to generate that exact same image. 

 
In [Figure 4](#fig:true_fcgr) is the fCGR for *E. coli*, for frequency 3. The relative amount of each 3-mer in that sequence is represented by the darkness of that box. In this fCGR, the most and least frequent 3-mers were denoted with white text. This image corresponds to the fCGR for *E. coli*, for frequency 3. 

<p align="center">
  <img id="fig:true_fcgr" src="https://github.com/user-attachments/assets/917672ae-a260-474b-95e2-34e5321aa665" width="80%">
  <br>
  <em><strong>Figure 4:</strong> Ground truth fCGR (k = 3), three most and least occurring 3-mers labelled.</em>
</p>

We can actually create a similar map for each of the models, using their top 1 predictions. Ideally, even if the models aren’t predicting the right 3-mer each time, they might be able to recreate this image, meaning that they have learned about the global distribution of the 3-mers. We did this, as shown in [Figure 5](#fig:pred_fcgrs). As we can see, all of the models did poorly on this. However, the CNN, and especially the Markov model, did the worst. 

<p align="center">
  <img id="fig:pred_fcgrs" src="https://github.com/user-attachments/assets/556080bc-75b0-4c41-963d-88406ce5f7f4" width="80%">
  <br>
  <em><strong>Figure 5:</strong> The fCGRs (k = 3) formed from the predictions of the four models, three most and least occurring 3-mers labelled.</em>
</p>

We then subtracted the ground truth map from the prediction maps to yield these difference map. Here, a lighter colour is better, as these difference maps capture error, with green representing overpredictions and red representing underpredictions. These are shown in [Figure 6](#fig:diff_fcgrs). We also turned these maps into values by taking the normalized sum of the absolute values in each box, with 1 being the worst, and 0 being perfect. These are shown by the “SAE” (sum of absolute error) values at the top figure.

<p align="center">
  <img id="fig:diff_fcgrs" src="https://github.com/user-attachments/assets/bdfb7fe8-6515-4fa5-ba3b-e877f2d7d3d6" width="80%">
  <br>
  <em><strong>Figure 6:</strong> Difference maps between the ground truth and fCGR (k = 3) and the fCGRs formed from model predictions, three most overpredicted and three most underpredicted 3-mers labelled.</em>
</p>

As for the limitations of our work (and thus, some of the areas where we plan to extend our study):
*	We only used a single genome: Using a variety of genomes in training could yield better results, even with respect to testing within a single sequence [12]
*	We did not use any pre-trained models, nor did we employ transfer-learning: Large pretrained models exist for next letter prediction in DNA [14, 15]. It would absolutely be worth expanding our next-*k*-mer ranking study to include these, however, when doing this, it will require a careful selection of the test set (to avoid any models being tested on material they saw in pre-training).
*	We limited the targets to *k*=3 for the *k*-mers: Larger values of *k* would yield a more useful end result, though it would more computationally expensive (as the number of classes scales exponentially with *k*).
*	We only investigated single step prediction: Autoregression would be interesting to explore, as a model that performs well at one-step 3-mer ranking may do poorly at multiple steps.
*	We limited our evaluation metrics to statistical evaluations: In terms of being able to apply this work in downstream applications (such as de Bruijn graph assembly, for branch pruning), it would be worth testing the performance of models within an actual applied setting. It would also be worth investigating biologically-inspired metrics (such as even simply looking at whether errors are concentrated to specific genomic regions).

# References

**References cited in our paper**:

[1] Scott Hotaling, Joanna L. Kelley, and Paul B. Frandsen. 2021. Toward a genome sequence for every animal: Where are we now? Proceedings of the National Academy of Sciences 118, 52 (2021), e2109019118. \
[2] Gerardo Ceballos, Paul R. Ehrlich, Anthony D. Barnosky, Andrés García, Robert M. Pringle, et al. 2015. Accelerated modern human–induced species losses: Entering the sixth mass extinction. Science Advances 1, 5 (2015), e1400253. \
[3] Liang Kuo-ching and Sakakibara Yasubumi. 2021. MetaVelvet-DL: a MetaVelvet deep learning extension for de novo metagenome assembly. BMC Bioinformatics 22 (2021), Suppl 6. \
[4] Josephine B Oehler, Helen Wright, Zornitza Stark, Andrew J Mallett, and Ulf Schmitz. 2023. The application of long-read sequencing in clinical settings. Human genomics 17, 1 (2023), 73. \
[5] Nicola De Maio, Liam P. Shaw, Alasdair Hubbard, Sophie George, Nick Sanderson, et al. 2019. Comparison of long-read sequencing technologies in the hybrid assembly of complex bacterial genomes. Microbial Genomics 5, 9 (2019), 1–12. \
[6] Tim J Puchtler, Kerr Johnson, Rebecca N Palmer, Emma L Talbot, Lindsey A Ibbotson, et al. 2020. Single-molecule DNA sequencing of widely varying GC-content using nucleotide release, capture and detection in microdroplets. Nucleic Acids Research 48, 22 (2020), e132–e132. \
[7] Heena Satam, Kandarp Joshi, Upasana Mangrolia, Sanober Waghoo, Gulnaz Zaidi, et al. 2023. Next-generation sequencing technology: Current trends and advancements. Biology 12, 7 (2023), 997. \
[8] Yingxue Yang, Wenjie Du, Yanchun Li, Jiawei Lei, and Weihua Pan. 2025. Recent advances and challenges in de novo genome assembly. Genomics Communications 2 (2025). \
[9] Camille Moeckel, Manvita Mareboina, Maxwell Konnaris, Candace Chan, Ioannis Mouratidis, et al. 2024. A Survey of k-mer Methods and Applications in Bioinformatics. Computational and Structural Biotechnology Journal 23 (2024), 2289–2303. \
[10] Mihai Pop. 2009. Genome assembly reborn: recent computational challenges. Briefings in bioinformatics 10 (2009), 354–366. \ 
[11] Yanrong Ji, Zhihan Zhou, Han Liu, and Ramana V Davuluri. 2021. DNABERT: pre-trained Bidirectional Encoder Representations from Transformers model for DNA-language in genome. Bioinformatics 37, 15 (2021), 2112–2120. \
[12] Hugo Dalla-Torre, Liam Gonzalez, Javier Mendoza Revilla, Nicolas Lopez Carranza, Adam Henryk Grywaczewski, et al. 2025. Nucleotide Transformer: Building and Evaluating Robust Foundation Models for Human Genomics. Nature Methods 22 (2025), 287–297. Issue 2. \
[13] Zhihan Zhou, Yanrong Ji, Weijian Li, Pratik Dutta, Ramana Davuluri, et al. 2024. DNABERT-2: Efficient Foundation Model and Benchmark For Multi-Species Genomes. In International Conference on Learning Representations, B. Kim, Y. Yue, S. Chaudhuri, K. Fragkiadaki, M. Khan, et al. (Eds.), Vol. 2024. 41642–41665. \
[14] Eric Nguyen, Michael Poli, Marjan Faizi, Armin W. Thomas, Callum Birch Sykes, et al. 2023. HyenaDNA: long-range genomic sequence modeling at single nucleotide resolution. In Proceedings of the 37th International Conference on Neural Information Processing Systems (NIPS ’23). Curran Associates Inc., Red Hook, NY, USA, Article 1872, 25 pages. \
[15] Eric Nguyen, Michael Poli, Matthew G. Durrant, Brian Kang, Dhruva Katrekar, et al. 2024. Sequence modeling and design from molecular to genome scale with Evo. Science 386, 6723 (2024), eado9336. \
[16] Andrey Kislyuk, Srijak Bhatnagar, Jonathan Dushoff, and Joshua S. Weitz. 2009. Unsupervised statistical clustering of environmental shotgun sequences. BMC Bioinformatics 10, 1 (2009), 316. \
[17] Väinö Jääskinen, Ville Parkkinen, Lu Cheng, and Jukka Corander. 2014. Bayesian clustering of DNA sequences using Markov chains and a stochastic partition model. Statistical Applications in Genetics and Molecular Biology 13, 1 (2014), 105–121. \
[18] Byung-Jun Yoon and Palghat P. Vaidyanathan. 2006. Context-Sensitive Hidden Markov Models for Modeling Long-Range Dependencies in Symbol Sequences. IEEE Transactions on Signal Processing 54, 11 (2006), 4169–4184. \
[19] Babak Alipanahi, Andrew Delong, Matthew Weirauch, and Brendan Frey. 2015. Predicting the sequence specificities of DNA- and RNA-binding proteins by deep learning. Nature biotechnology 33 (2015), 831–838. \
[20] Jian Zhou and Olga Troyanskaya. 2015. Predicting effects of noncoding variants with deep learning-based sequence model. Nature methods 12 (2015), 931–934. \
[21] Sepp Hochreiter and Jürgen Schmidhuber. 1997. Long Short-Term Memory. Neural Computation 9, 8 (1997), 1735–1780. \
[22] Byunghan Lee, Taehoon Lee, Byunggook Na, and Sungroh Yoon. 2015. DNA-Level Splice Junction Prediction using Deep Recurrent Neural Networks. arXiv preprint (2015), arXiv:1512.05135. \
[23] Eric W Sayers, Jeffrey Beck, Evan E Bolton, Devon Bourexis, James R Brister, et al. 2020. Database resources of the National Center for Biotechnology Information. Nucleic Acids Research 49, D1 (2020), D10–D17. \
[24] Natividad Ruiz and Thomas J. Silhavy. 2022. How Escherichia coli Became the Flagship Bacterium of Molecular Biology. Journal of Bacteriology 204, 9 (2022), e00230–e00222. \
[25] H. Joel Jeffrey. 1990. Chaos game representation of gene structure. Nucleic Acids Research 18, 8 (1990), 2163–2170. \
[26] Susana Vinga, Alexandra Carvalho, Alexandre Francisco, Luís Russo, and Jonas Almeida. 2012. Pattern matching through chaos game representation: Bridging numerical and discrete data structures for biological sequence analysis. Algorithms for molecular biology : AMB 7 (2012), 10.

**References used only in this explanation**:

[27] Jasper Fuk-Woo Chan, Shuofeng Yuan, Kin-Hang Kok, Kelvin Kai-Wang To, Hin Chu, et al. A familial cluster of pneumonia associated with the 2019 novel coronavirus indicating person-to-person transmission: a study of a family cluster. Lancet 395, 10223 (2020), 514–523. \
[28] Çiğdem Köroğlu, Peng Chen, Michael Traurig, Serdar Altok, Clifton Bogardus, et al. De Novo Genome Assemblies From Two Indigenous Americans from Arizona Identify New Polymorphisms in Non-Reference Sequences. Genome Biology and Evolution 16, 9 (2024), evae188. \
[29] Stephan Solis-Reyes. DNA sequence classification: It’s easier than you think: An open-source k-mer based machine learning tool for fast and accurate classification of a variety of genomic datasets. Master’s thesis, The University of Western Ontario (2018), 5792.

### Q&A:

**Question**: Could you go over the Top-*N* accuracy and rank a bit more?

**Answer**: When we talk about Top-*N* accuracy, we’re talking about how often a model had the correct label for a datapoint within its Top *N* guesses. So for the overlapping 3-mers, 58.9% of the time across all folds, the transformer managed to get the correct prediction within its top 20 predictions. The rank refers to where, on average, the true prediction was found among the potential candidates predicted by the model.

But I think a better way to explain all of that is with an example: imagine a game, where I think of a number between 1 and 100, and you have five guesses to get there (also, you get a better score if you guess it in your earlier guesses, rather than getting it on the fourth or fifth guess). Since this game is so fun, we play this five times (willing suspension of disbelief and all).

| My number | Guess #1 | Guess #2 | Guess #3 | Guess #4 | Guess #5 |
| --------- | -------- | -------- | -------- | -------- | -------- |
| 22        | 1        | 100      | 50       | 25       | 75       |
| 1         | 22       | 100      | 50       | 25       | 75       |
| 94        | 1        | 22       | 95       | 94       | \-       |
| 1         | 1        | \-       | \-       | \-       | \-       |
| 63        | 1        | 22       | 63       | \-       | \-       |

You managed to win 3 games, of the 5 games in total: thus, you obtained a top-5 accuracy of 60% (meaning you got the correct answer within 5 guesses 60% of the time). 
On game 3, you got it on your 4th guess (rank 4). On the 4th game, in the first guess (rank 1). On the last game, on your 3rd guess (rank 3).
Since you got the correct answer in 3 guesses or less in two games, you got a top-3 accuracy of 40%. You only managed to get the correct answer on the first guess in one game, so your top-1 accuracy is 20%.

**Question**: So predict 3-mers? Why not 4-mers, or 1000-mers?

**Answer**: This has everything to do with the scope of this project. As an extension of our work, we definitely would like to look at ranking *k*-mers for other values of *k*. However, for this project, we limited ourselves to 3-mers, as the number of potential *k*-mers scales exponentially with *k*. Specifically, there exist 4^*k* classes for this classification problem.

As for why we chose 3-mers specifically (other than after some testing, we found it to be the highest value of *k* that our models could handle, while still being able to train fast enough to allow us to try new iterations of our experiments): when building proteins (amino acid chains) DNA is transcribed to mRNA, and mRNA is translated to amino acids, where each group of 3 letters in the mRNA states what amino acid is to be used. Thus, using groups of 3 letters seemed like a reasonable choice.

**Question**: You mention 3-mers, but you’re training machine learning models, which means they need arrays of numbers as input. How did you encode the 3-mers numerically?

**Answer**: We map sequences to base-4 values (A=0, C=1, G=2, T=3) and calculate a unique integer for each 3-mer, scaling left-to-right by increasing powers of 4 (i.e., AAA = 0, CAA = 1, AAC = 16). Rather than looking at the 99 letter datapoints used in this study, let’s look at a 12 letter example: AAGCCAGTGACC. Here, AAC is the label, and AAGCCAGTG is the context. The below table explains how this gets tokenized according to the three tokenization schemes, and the numerical encoding for each:

| Tokenization scheme     | Tokens                              | Numerical encodings         |
| ----------------------- | ----------------------------------- | --------------------------- |
| Non-overlapping 3-mers: | [AAG, CCA, GTG]                     | [32, 5, 46]                 |
| Overlapping 3-mers:     | [AAG, AGC, GCC, CCA, CAG, AGT, GTG] | [32, 24, 22, 5, 33, 56, 46] |
| Single nucleotides:     | [A, A, G, C, C, A, G, T, G]         | [0, 0, 2, 1, 1, 0, 2, 3, 2] |

Note that, given the 96-letter contexts used by the models, that means that the single nucleotide scheme creates a context of 96 tokens, the overlapping 3-mer scheme creates 94, and the non-overlapping 3-mer scheme creates 32.

**Question**: So, overall, the overlapping 3-mer tokenization scheme works best?

**Answer**: For what we were specifically measuring (Top-*N* accuracy), **yes**. But, it’s a bit more complicated than that. For one, while we noticed that the transformer performed best with the overlapping 3-mers, the simpler (and thus faster-to-train) LSTM was the best performing model for the two other tokenization schemes. Also, since our main metric was Top-*N* accuracy, and since the overlapping 3-mer scheme led to the best results in this respect, we then focussed further on that tokenization scheme. However, in extending our work, we plan to further investigate the non-overlapping scheme, as it led to nearly as good results, in a fraction of the time. The following table is the time (in seconds) taken to train the three neural models for a single fold:

| Model       | Single | Overlapping | Non-overlapping |
| ----------- | ------ | ----------- | --------------- |
| Transformer | 7473   | 6940        | 727             |
| LSTM        | 1043   | 974         | 162             |
| CNN         | 427    | 494         | 89              |

So, for this study, yes, the overlapping 3-mer tokenization scheme works best, but the non-overlapping 3-mer tokenization scheme shows a lot of potential for getting similar results, using a fraction of the computational resources.
