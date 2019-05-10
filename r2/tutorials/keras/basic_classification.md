
# 训练您的第一个神经网络：基本分类

<table class="tfo-notebook-buttons" align="left">
  <td>
    <a target="_blank" href="https://www.tensorflow.org/alpha/tutorials/keras/basic_classification"><img src="https://www.tensorflow.org/images/tf_logo_32px.png" />View on TensorFlow.org</a>
  </td>
  <td>
    <a target="_blank" href="https://colab.research.google.com/github/tensorflow/docs/blob/master/site/en/r2/tutorials/keras/basic_classification.ipynb"><img src="https://www.tensorflow.org/images/colab_logo_32px.png" />Run in Google Colab</a>
  </td>
  <td>
    <a target="_blank" href="https://github.com/tensorflow/docs/blob/master/site/en/r2/tutorials/keras/basic_classification.ipynb"><img src="https://www.tensorflow.org/images/GitHub-Mark-32px.png" />View source on GitHub</a>
  </td>
</table>

本指南训练神经网络模型，对服装图像进行分类，如运动鞋和衬衫。如果您不了解所有细节，那也没关系;
这是一个完整的TensorFlow程序的快节奏概述，详细解释了我们的目标。

本指南使用[tf.keras](https://www.tensorflow.org/guide/keras)，一个高级API，用于在TensorFlow中构建和训练模型。
安装

```
pip install tensorflow==2.0.0-alpha0
```

导入相关库

```
from __future__ import absolute_import, division, print_function, unicode_literals

# TensorFlow and tf.keras
import tensorflow as tf
from tensorflow import keras

# Helper libraries
import numpy as np
import matplotlib.pyplot as plt

print(tf.__version__)
```

## 导入MNIST数据集

本指南使用[Fashion MNIST](https://github.com/zalandoresearch/fashion-mnist) 数据集，该数据集包含10个类别中的70,000个灰度图像。 图像显示了低分辨率（28 x 28像素）的单件服装，如下所示：

<table>
  <tr><td>
    <img src="https://tensorflow.google.cn/images/fashion-mnist-sprite.png"
         alt="Fashion MNIST sprite"  width="600">
  </td></tr>
  <tr><td align="center">
    <b>Figure 1.</b> <a href="https://github.com/zalandoresearch/fashion-mnist">Fashion-MNIST samples</a> (by Zalando, MIT License).<br/>&nbsp;
  </td></tr>
</table>

Fashion MNIST旨在替代经典的MNIST数据集 - 通常用作计算机视觉机器学习计划的“Hello，World”。MNIST数据集包含手写数字（0,1,2等）的图像，其格式与我们将在此处使用的服装的格式相同。

本指南使用Fashion MNIST的多样性，因为这是一个比普通 [MNIST](http://yann.lecun.com/exdb/mnist/)稍微更具挑战性的问题。这两个数据集都相对较小，用于验证算法是否按预期工作。它们是测试和调试代码的良好起点。

我们将使用60,000张图像来训练网络，10,000张图像来评估网络模型的准确度。您可以直接从TensorFlow访问Fashion MNIST，直接从TensorFlow导入并加载Fashion MNIST数据：

```
fashion_mnist = keras.datasets.fashion_mnist

(train_images, train_labels), (test_images, test_labels) = fashion_mnist.load_data()
```

加载数据返回4个NumPy数组：

* `train_images`和`train_labels`数组是*训练集* – 模型用于学习的数据。
* 模型将根据测试集`test_images`和`test_labels`数组进行测试。

图像为28x28 NumPy数组，像素值范围为0到255，标签是一个整数数组，范围为0到9，这些对应于图像所代表的服装类别：

<table>
  <tr>
    <th>Label</th>
    <th>Class</th>
  </tr>
  <tr>
    <td>0</td>
    <td>T-shirt/top</td>
  </tr>
  <tr>
    <td>1</td>
    <td>Trouser</td>
  </tr>
    <tr>
    <td>2</td>
    <td>Pullover</td>
  </tr>
    <tr>
    <td>3</td>
    <td>Dress</td>
  </tr>
    <tr>
    <td>4</td>
    <td>Coat</td>
  </tr>
    <tr>
    <td>5</td>
    <td>Sandal</td>
  </tr>
    <tr>
    <td>6</td>
    <td>Shirt</td>
  </tr>
    <tr>
    <td>7</td>
    <td>Sneaker</td>
  </tr>
    <tr>
    <td>8</td>
    <td>Bag</td>
  </tr>
    <tr>
    <td>9</td>
    <td>Ankle boot</td>
  </tr>
</table>

每个图像都映射到一个标签，由于类名不包含在数据集中，因此将它们存储在此处以便在绘制图像时使用：

```
class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat',
               'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']
```

##  探索数据

让我们在训练模型之前探索数据集的格式,以下显示训练集中有60,000个图像，每个图像表示为28 x 28像素：

```
train_images.shape
```

`(60000, 28, 28)`

同样，训练集中有60,000个标签：

```
len(train_labels)
```

`60000`

每个标签都是0到9之间的整数：

```
train_labels
```

`array([9, 0, 0, ..., 3, 0, 5], dtype=uint8)`

测试集中有10,000个图像。同样，每个图像表示为28 x 28像素：

```
test_images.shape
```

(10000, 28, 28)

测试集包含10,000个图像标签：

```
len(test_labels)
```
10000

## 预处理数据

在训练网络之前必须对数据进行预处理。 如果您检查训练集中的第一个图像，您将看到像素值落在0到255的范围内：

```
plt.figure()
plt.imshow(train_images[0])
plt.colorbar()
plt.grid(False)
plt.show()
```
![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_21_0.png)

在将它们输入到神经网络模型之前，我们将这些值缩放到0到1的范围。 为此，我们将值除以255。重要的是*训练集*和*测试集*以相同的方式进行预处理：

```
train_images = train_images / 255.0

test_images = test_images / 255.0
```

为了验证数据的格式是否正确以及我们是否已准备好构建和训练网络，让我们显示训练集中的前25个图像，并在每个图像下方显示类名。

```
plt.figure(figsize=(10,10))
for i in range(25):
    plt.subplot(5,5,i+1)
    plt.xticks([])
    plt.yticks([])
    plt.grid(False)
    plt.imshow(train_images[i], cmap=plt.cm.binary)
    plt.xlabel(class_names[train_labels[i]])
plt.show()
```
![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_25_0.png)

## 构建模型

构建神经网络需要配置模型的层，然后编译模型。

### 设置图层

神经网络的基本构建块是 *layer*，图层从提供给它们的数据中提取表示，希望这些表示对于手头的问题是有意义的。

大多数深度学习包括将简单层链接在一起。大多数图层（例如`tf.keras.layers.Dense`）都具有在训练期间学习到的参数。


```
model = keras.Sequential([
    keras.layers.Flatten(input_shape=(28, 28)),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dense(10, activation='softmax')
])
```


该网络中的第一层`tf.keras.layers.Flatten`将图像的格式从二维数组（28 x 28像素）转换为一维数组（28 * 28 = 784像素））。可以将此图层视为图像中未堆叠的像素行并将它们排列在一起。该层没有要学习的参数,它只重新格式化数据。

在像素被展平之后，网络由两个tf.keras.layers.Dense层的序列组成。这些是密集连接或完全连接的神经层。第一个Dense层有128个节点（或神经元）。第二层是10节点softmax层，其返回10个概率分数的数组，其总和为1.每个节点包含指示当前图像属于10个类之一的概率的分数。
在像素被展平之后，网络由两个`tf.keras.layers.Dense`层的序列组成。这些是密集连接或完全连接的神经层。第一个`Dense`层有128个节点（或神经元）。第二（最后）层是10节点*softmax*层，其返回10个概率分数的数组，其总和为1.每个节点包含指示当前图像属于10个类别之一的概率的分数。

### 编译模型

在模型准备好进行训练之前，它需要更多设置。这些是在模型的 *compile* 步骤中添加的

* 损失函数，可以衡量模型在训练过程中的准确程度。我们希望最小化此函数，以便在正确的方向上“引导”模型。
* 优化器，这是基于它看到的数据及其损失函数更新模型的方式。
* 度量标准，用于监控训练和测试步骤。以下示例使用精度，即正确分类的图像分数。

```
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
```

## 训练模型

训练神经网络模型需要以下步骤：

1. 将训练数据输入模型。在此实例中，训练数据位于`train_images`和`train_lables`数组中。
2. 该模型学习如何关联图像和标签。
3. 我们要求模型对测试集进行预测，在本例中为`test_images`数组。我们验证预测是否与`test_labels`数组中的标签匹配。

要开始训练，请调用 `model.fit` 方法，因为它将模型“fits拟合”到训练数据：

```
model.fit(train_images, train_labels, epochs=5)
```

随着模型训练，显示损失和准确度指标。该模型在训练数据上达到约0.88（或88％）的准确度。


## 评估精度

接下来，比较模型在测试数据集上的表现情况：

```
test_loss, test_acc = model.evaluate(test_images, test_labels)

print('\nTest accuracy:', test_acc)
```

事实证明，测试数据集的准确性略低于训练数据集的准确性。训练精度和测试精度之间的差距表示过度拟合(*overfitting*)。过度拟合是指机器学习模型在新的数据比在训练数据上表现更差，也就是泛化性不好。


## 预测

通过训练模型，我们可以使用它来预测某些图形。

```
predictions = model.predict(test_images)
```

这里，模型已经预测了测试集中每个图像的标签，我们来看看第一个预测：

```
predictions[0]
```
输出：
```
array([6.2482708e-05, 2.4860196e-08, 9.7165821e-07, 4.7436039e-08,
       2.0804382e-06, 1.3316551e-02, 9.8731316e-06, 3.4591161e-02,
       1.2390658e-04, 9.5189297e-01], dtype=float32)
```       

预测是10个数字的数组，它们代表模型的“confidence（置信度）”，即图像对应于10种不同服装中的每一种。我们可以看到哪个标签具有最高的置信度值：

```
np.argmax(predictions[0])
```
`9`

因此，模型最确信这是一个ankle boot(踝靴)，或class_names[9]。检查测试标签，该分类是正确的：

```
test_labels[0]
```
`9`

我们可以通过图表来查看全部10个通道：

```
def plot_image(i, predictions_array, true_label, img):
  predictions_array, true_label, img = predictions_array[i], true_label[i], img[i]
  plt.grid(False)
  plt.xticks([])
  plt.yticks([])

  plt.imshow(img, cmap=plt.cm.binary)

  predicted_label = np.argmax(predictions_array)
  if predicted_label == true_label:
    color = 'blue'
  else:
    color = 'red'

  plt.xlabel("{} {:2.0f}% ({})".format(class_names[predicted_label],
                                100*np.max(predictions_array),
                                class_names[true_label]),
                                color=color)

def plot_value_array(i, predictions_array, true_label):
  predictions_array, true_label = predictions_array[i], true_label[i]
  plt.grid(False)
  plt.xticks([])
  plt.yticks([])
  thisplot = plt.bar(range(10), predictions_array, color="#777777")
  plt.ylim([0, 1])
  predicted_label = np.argmax(predictions_array)

  thisplot[predicted_label].set_color('red')
  thisplot[true_label].set_color('blue')
```

让我们看看第0个图像，预测和预测数组。

```
i = 0
plt.figure(figsize=(6,3))
plt.subplot(1,2,1)
plot_image(i, predictions, test_labels, test_images)
plt.subplot(1,2,2)
plot_value_array(i, predictions,  test_labels)
plt.show()
```

![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_48_0.png)

```
i = 12
plt.figure(figsize=(6,3))
plt.subplot(1,2,1)
plot_image(i, predictions, test_labels, test_images)
plt.subplot(1,2,2)
plot_value_array(i, predictions,  test_labels)
plt.show()
```

![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_49_0.png)

让我们用他们的预测绘制几个图像。正确的预测标签是蓝色的，不正确的预测标签是红色的，该数字给出了预测标签的百分比（满分100）。请注意，即使非常自信，模型也可能出错。


```
# 绘制前X个测试图像，预测标签和真实标签。 
# 用蓝色标记正确的预测，用红色标记错误的预测。
num_rows = 5
num_cols = 3
num_images = num_rows*num_cols
plt.figure(figsize=(2*2*num_cols, 2*num_rows))
for i in range(num_images):
  plt.subplot(num_rows, 2*num_cols, 2*i+1)
  plot_image(i, predictions, test_labels, test_images)
  plt.subplot(num_rows, 2*num_cols, 2*i+2)
  plot_value_array(i, predictions, test_labels)
plt.show()
```

![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_51_0.png)

最后，使用训练的模型对单个图像进行预测。

```
# 从测试数据集中获取图像
img = test_images[0]

print(img.shape)
```

`tf.keras`模型经过优化，可以立即对*批量*或集合进行预测。因此，即使我们使用单个图像，我们也需要将其添加到列表中：

```
# 将图像添加到批次中，它是唯一的成员。 
img = (np.expand_dims(img,0))

print(img.shape)
```

`(1, 28, 28)`

现在预测此图像的正确标签：

```
predictions_single = model.predict(img)

print(predictions_single)
```


```
plot_value_array(0, predictions_single, test_labels)
_ = plt.xticks(range(10), class_names, rotation=45)
```

![](https://tensorflow.google.cn/alpha/tutorials/keras/basic_classification_files/output_58_0.png)


`model.predict`返回列表清单- 一批数据中每个图像的列表，抓取批次中我们（唯一）图像的预测：
```
np.argmax(predictions_single[0])
```

`9`

和前面的一样，模型预测标签为9。