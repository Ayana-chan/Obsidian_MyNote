# 生词 & 关键词

iterative 迭代的
camouflflage 伪装，迷彩
interaction 交互
artefacts 伪影
segmentation 分割
monochrome 黑白的
opacity 透明度

made two enhancements to the graph cuts mechanism:
- iterative estimation: 进行一次之后有了一定的前景背景差别，第二次进行会更加准确。
- incomplete labelling: 用户只标记Tb。

trimap: 三分图，分为前景Tf、背景Tb、未知灰色区域Tu

# 后备知识

## MLE 最大似然估计

求取一种高斯分布，使得所有样本同时发生的概率最大。取log是为了好拆式子，但GMM的话由于log里面还有连加，拆不出来。

## EM 期望最大算法

一个一直用到的公式：

$$p(x|θ)=\frac{p(x,z|θ)}{p(z|x,θ)}$$

每次迭代，$θ^t$都可看做常量。

- E-Step: 把$Q(θ,θ^t)$（argmax右边的式子）给列出来。
- M-Step: 求解最大值和argmax

## GMM 高斯混合模型

若有一堆样本，我们猜测它是由两个高斯分布组成的，那么我们就需要猜测它属于哪个高斯分布。这个归属z就称作隐变量latent variable。

样本x为观测数据observed variable。

二者结合的(x,z)为完整数据complete data。

而θ


# 小记

最小割问题等价于最大流问题。

以S和T作为源点和终点，做最小割之后，即可用最小的代价将背景和前景分开。

argmin: 代入指定的参数的所有可能值，返回所得的最小结果所代入的参数。

![400](assets/uTools_1683973454372.png)

- t-link: 与S、T相连的边的权值。
- n-link: 图像中相邻像素之间的边的权值。
## Graph Cut

- z: 图像数据
- α: 对像素的判定结果
- θ: 分布模型，或者说参数

E(α,θ,z) = U(α,θ,z) + V(α,z)


- U: the fit of the opacity distribution α to the data z
- V: smoothness term

- C: the set of pairs of neighboring pixels
- dis(·): the Euclidean distance of neighbouring pixels.
- <·>: expectation over an image sample
- β: 图片整体的对比度越低，β越大，从而能在低对比度图片下工作
- γ: 50 , a versatile setting for a wide variety of images

相邻（neighboring）为八向（上下左右对角皆可）。

- First, the monochrome image model is replaced for colour by a Gaussian Mixture Model (GMM) in place of histograms. 
- Secondly, the one-shot minimum cut estimation algorithm is replaced by a more powerful, iterative procedure that alternates between estimation and parameter learning. 
- Thirdly, the demands on the interactive user are relaxed by allowing incomplete labelling — the user specififies only TB for the trimap, and this can be done simply by placing a rectangle or a lasso around the object.

## GrabCut

Each GMM, one for the background and one for the foreground, is taken to be a full-covariance Gaussian mixture with K components (typically K = 5).具有K个分量的全协方差高斯混合

E(α,k,θ,z) = U(α,k,θ,z) + V(α,z)

In order to deal with the GMM tractably, in the optimization framework, an additional vector k = {k1,...,kn,...,kN} is introduced, with kn ∈ {1,...K}, assigning, to each pixel, a unique GMM component, one component either from the background or the foreground model, according as αn = 0 or 11 .

- K: 5 高斯分量数
- k、kn: 第n个像素对应哪个高斯分量，1~K。要不来自

式9的第二第三项是对高斯分布的正态密度求log后取负，涉及多元统计的知识。∑在这里表示方差。

混合高斯模型：

![](assets/uTools_1683950332223.png)

画矩形相当于初始化背景部分Tb。

？ Note that for effificiency the optimal flflow, computed by Graph Cut, can be re-used during user edits.

# 实现

穷举边时，虽然每个点都有八个邻居，但每个点只要算四个方向，就能覆盖所有邻居连线。例如，右上、右、右下、下。

可能要注意不把用户指定的前景背景给迭代更新了。










