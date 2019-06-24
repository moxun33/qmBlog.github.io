---
title: 手把手教你用 PHP 拼接微信好友头像
date: 2017-06-13 20:18:09
tags:
     - PHP
     - 微信
---
+  最近做个人微信机器人挺热门的， 同时很多人也用来对自己的微信好友做分析和统计， 比如：简单的就是利用微信好友的头像做成一张大图， 让朋友圈的好友都看到彼此， 觉得蛮有意思的! 网上已经有 Python 或 Nodejs 的实现了， 我就想用 PHP 来实现一下。
 <!-- more -->

### 1、如何选一个微信爬虫。

#### 所谓的微信爬虫就是利用微信的 web api 做一些自动化的事情，这里强烈推荐由 PHP7实现的[vbot](https://github.com/HanSon/vbot), 所有的东西都可以在 vbot 的文档里找到， 在微信 web api 的范围内你都可以为所欲为了。注意： 本文所有的微信数据都基于 vbot 获取的。

### 2、如何爬取微信好友的头像。

#### 首先要获取的是你的微信好友列表， 在 vbot 的基础上我们可以这样做：

```php
	File::saveTo(__DIR__.'/contacts.json', $contacts['friends']);
```
#### 把微信好友列表写入到一个文件中。
#### 然后对这个文件进行处理， 提取微信好友的头像链接和下载。

```php
  $contactAmount = $friends->count();
     for($i=0; $i< $contactAmount; $i++){
       $data =  $friends->getAvatar(self::$usernameArr[$i]);
      file_put_contents(__DIR__.'/avatars/'.$i.'.jpeg',$data);

     }
```
#### 就这样， 我们就已经能把所有的微信好友头像下载到指定的地方了。注意：建议图片的命名规则用数字递增的方式， 方便后面的图片拼接循环。

### 3、图片拼接
#### 本文的重点来了， 现在网上有很多图片拼接小工具， 但 PHP 实现的寥寥无几， 找了一轮后， 决定还是自己写。
（1）首先，要确定画布的长度和宽度。那怎么根据图片数量确定画布大小呢。我们门这样分析：由于微信头像的宽高相等， 那我们拼接到画布的托也是宽高相等，这里全部设置为200px， 那么， 只要确定行数和列数， 就知道画布的大小了。**还有一点注意：当图片数量小于4张时， 我们只需要一行（或者一列）即可。** 当图片数大于3时：

a、确定列数（每行多少张）：列数 = 总数（大于3）的平方根， 向上取整
b、确定行数（每列多少张）：总数（大于3）除以列数， 向上取整

* 这里的行数和行数的计算方式是为了尽可能使画布的宽度和高度差值减小。你也可以根据自己的好修改计算方法， 但一定要跟后面的图片排列的行数和列数一致， 否则会出现图片越界的问题。

知道了行数和列数， 那很容易得出画布的宽高  

c、画布宽度 = 图片宽度（200px）乘以 列数
d、画布高度 = 图片高度（200px）乘以 行数

代码如下：
```php
	 /**
     * 根据图片数据计算画布的大小
     * 默认每张图宽度高度 = 200
     * 画布大小计算方法：
     * 列数 = 总数（大于3）的平方根， 向上取整
     * 行数 = 总数（大于3）除以列数， 向上取整
     *  @param $imageCount
     */
    private function  prepareForCanvas($imageCount){
        $canvasWidth = 0;
        $canvasHeight = 0;
        if ($imageCount > 0) {

            //宽度 = 列数*200
            if ($imageCount < 4) {

                    $canvasWidth = $this->width*$imageCount;
                    $this->eachLineCount = $imageCount;
                    $canvasHeight = $this->height;
            }else{

                $this->eachLineCount = ceil(sqrt($imageCount)); //列数
                $this->eachColumCount = ceil($imageCount/$this->eachLineCount);
                $canvasWidth = $this->width * $this->eachLineCount;
                $canvasHeight = $this->height * $this->eachColumCount;
            }
        }
        echo '图片总数：'.$imageCount.PHP_EOL;
        //创建画布
        $this->createCanvas($canvasWidth, $canvasHeight);
    }

       /**创建画布
     * @param $width
     * @param $height
     */
    private function createCanvas($cwidth, $cheight) {
        $totalImage = count($this->srcImages);     

        $this->canvas = imagecreatetruecolor($cwidth, $cheight);

        // 使画布透明
        $white = imagecolorallocate($this->canvas, 255, 255, 255);
        imagefill($this->canvas, 0, 0, $white);
        imagecolortransparent($this->canvas, $white);
        echo '画布大小:长度：'.$cwidth.'，高度：'. $cheight.PHP_EOL.
            '每行数量（列数）：'.$this->eachLineCount.',每列数量（行数）：'.$this->eachColumCount.PHP_EOL;
    }
```

### 4、画布建好了， 接下来就是拼接图片了。
#### 这里的难点在于确定图片在哪个位置。 

a、图片数小于4时， 只有一行， 每张图片的 y 值都是0， x 值随图片数每次递增200px。
b、图片数大于3时， 循环复制图片到画布上， 当图片的序号等于列数的倍数时， 就要换行， 那么 x 值从0开始， y 值递增200px； 以此类推，直到所有图片复制到画布上。

代码如下：
```php
    /**
     * 合并图片
     */
    public function combine() {
        if (empty($this->srcImages)  || $this->width==0 || $this->height==0) {
            return;
        }
        $imageCount = count($this->srcImages);
        $this->prepareForCanvas($imageCount);
        for($i=0; $i< $imageCount; $i++) {
            $srcImage = $this->srcImages[$i];
            $srcImageInfo = getimagesize($srcImage);
            // 如果能够正确的获取原图的基本信息
            if ($srcImageInfo) {
                $srcWidth = $srcImageInfo[0];
                $srcHeight = $srcImageInfo[1];
                $fileType = $srcImageInfo[2];
                if ($fileType == 2) {
                    // 原图是 jpg 类型
                    $srcImage = imagecreatefromjpeg($srcImage);
                } else if ($fileType == 3) {
                    // 原图是 png 类型
                    $srcImage = imagecreatefrompng($srcImage);
                } else {
                    // 无法识别的类型
                    continue;
                }

                //只支持横向
                // 计算当前原图片应该位于画布的哪个位置
                if ($i < $this->eachLineCount){
                    $destX = $i * $this->width;
                    $desyY = 0;
                    $currentRowIndex = 0;
                }else{
                    //计算该图片应该在第几行， 第几列
                     $tmp = ($i+1)/$this->eachLineCount;
                    if (($i+1)%$this->eachLineCount == 0){
                        $currentRowIndex = $tmp -1;
                    }else{
                        $currentRowIndex = floor($tmp);
                    }


                    $destX = ($i - $currentRowIndex*$this->eachLineCount)*$this->width;
                    $desyY = $currentRowIndex*$this->height;
                }

                echo '当前索引:'.$i.',当前行的索引: '.$currentRowIndex.',图片位置 X: '.$destX.',图片位置 Y: '.$desyY.PHP_EOL;

                imagecopyresampled($this->canvas, $srcImage, $destX, $desyY,
                    0, 0, $this->width, $this->height, $srcWidth, $srcHeight);
            }
        }

        // 如果有指定目标地址，则输出到文件
        if ( ! empty($this->destImage)) {
            $this->output();
        }
    }

```

### 5、效果预览

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fgfalih389j30b40b40t8.jpg)

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fgfalhd9z8j30rs0m8acd.jpg)

### 6、完整的代码
[https://github.com/moxun33/imageMergerUtil](https://github.com/moxun33/imageMergerUtil)




