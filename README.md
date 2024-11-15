

# 前言




  对于图像拼接，前面探讨了通过基于Stitcher进行拼接过渡和基于特征点进行拼接过渡，这2个过渡的方式是摄像头拍摄角度和方向不应差距太大。  对于特定的场景，本身摄像头拍摄角度差距较大，拉伸变换后也难做到完美的缝隙拼接，这个时候使用渐近过渡反倒是最好的。



 

# Demo




  单独蒙版   ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212924-1022728551.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213225-1603499807.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213123-1815368493.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213121-339590481.png)




  蒙版过渡，这里只是根据图来，其实可对每个像素对于第一张图为系数k，而第二张为255\-k，实现渐近过渡。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213311-1717117972.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212989-893345895.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213084-1584622033.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213175-320734159.png)




  直接使用第一张蒙版优化  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212999-526118793.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213211-1662052794.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213107-1883465510.png)



 

# 准本蒙版




  蒙版可以混合，也可以分开，为了让读者更好的深入理解原理，这里都使用：  找个工具，造单色渐进色，红色蒙版，只是r通道，bga都为0  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213010-1013601934.png)




  （注意：使用rgba四通道）  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212901-1298656515.png)




  （上面这张图，加了边框，导致了“入坑二”打印像素值不对）  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213041-12251468.png)




  由于工具渐进色无法叠层，这个工具无法实现rgba不同向渐进色再一张图（横向、纵向、斜向），更改了方式，每个使用一张图：  为了方便，不管a通道了，直接a为100%（255）。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213098-570531184.png)




  再弄另外一个通道的：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213097-1213148379.png)




  在这里使用工具就只能单独一张了：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212962-752377787.png)



 

# 一个蒙版图的过渡实例




## 步骤一：打开图片和蒙版




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213102-214723003.png)





```
   cv::Mat matLeft = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/29.jpg");
    cv::Mat matRight = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/30.jpg");
    cv::Mat matMask1 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/37.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask2 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/38.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask3 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/39.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask4 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/40.png", cv::IMREAD_UNCHANGED);

```



## 步骤二：将蒙版变成和原图一样大小




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213350-1078300160.png)





```
    cv::resize(matLeft, matLeft, cv::Size(0, 0), 0.5, 0.5);
    cv::resize(matRight, matRight, cv::Size(0, 0), 0.5, 0.5);
    cv::resize(matMask1, matMask1, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask2, matMask2, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask3, matMask3, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask4, matMask4, cv::Size(matLeft.cols, matLeft.rows));

```



## 步骤三：底图




  由于两张图虽然是同样大小，但是其不是按照整体拼接后的大小，所以需要假设一个拼接后的大小的底图。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213120-1150540970.png)





```
    // 底图，扩大500横向，方便移动
    cv::Mat matResult = cv::Mat(matLeft.rows, matLeft.cols + 500, CV_8UC3);

```



## 步骤四：原图融合




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213112-375365635.png)





```
        // 副本，每次都要重新清空来调整
        cv::Mat matResult2 = matResult.clone();
#if 1
        // 第一张图，直接比例赋值，因为底图为0
        for(int row = 0; row < matLeft.rows; row++)
        {
            for(int col = 0; col < matLeft.cols; col++)
            {
                double r = matMask1.at(row, col)[2] / 255.0f;
//                double r = matMask2.at(row, col)[1] / 255.0f;
//                double r = matMask3.at(row, col)[0] / 255.0f;
//                double r = matMask4.at(row, col)[0] / 255.0f;
                matResult2.at(row, col)[0] = (matLeft.at(row, col)[0] * r);
                matResult2.at(row, col)[1] = (matLeft.at(row, col)[1] * r);
                matResult2.at(row, col)[2] = (uchar)(matLeft.at(row, col)[2] * r);
            }
        }
#endif

```



## 步骤五：另外一张图的融合




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213323-419998208.png)





```
#if 1
        // 第二张图，加法，因为底图为原图了
        for(int row = 0; row < matRight.rows; row++)
        {
            for(int col = 0; col < matRight.cols; col++)
            {
                double g = matMask2.at(row, col)[1] / 255.0f;
                // 偏移了x坐标
                matResult2.at(row, col + x)[0] += matRight.at(row, col)[0] * g;
                matResult2.at(row, col + x)[1] += matRight.at(row, col)[1] * g;
                matResult2.at(row, col + x)[2] += matRight.at(row, col)[2] * g;
            }
        }
#endif

```



## 步骤六（与步骤五互斥）：优化的融合




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213337-1576636237.png)





```
#if 1
        // 第二张图，加法，因为底图为原图了（优化）
        for(int row = 0; row < matRight.rows; row++)
        {
            for(int col = 0; col < matRight.cols; col++)
            {
                double r2;
                if(x + col <= matLeft.cols)
                {
                    r2 = (255 - matMask1.at(row, col + x)[2]) / 255.0f;
                }else{
                    r2 = 1.0f;
                }
                // 偏移了x坐标
                matResult2.at(row, col + x)[0] += matRight.at(row, col)[0] * r2;
                matResult2.at(row, col + x)[1] += matRight.at(row, col)[1] * r2;
                matResult2.at(row, col + x)[2] += matRight.at(row, col)[2] * r2;
            }
        }
#endif

```


 

# 函数原型




  手码的像素算法，没有什么高级函数。



 

# Demo源码





```
void OpenCVManager::testMaskSplicing()
{
    cv::Mat matLeft = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/29.jpg");
    cv::Mat matRight = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/30.jpg");
    cv::Mat matMask1 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/37.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask2 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/38.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask3 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/39.png", cv::IMREAD_UNCHANGED);
    cv::Mat matMask4 = cv::imread("D:/qtProject/openCVDemo/openCVDemo/modules/openCVManager/images/40.png", cv::IMREAD_UNCHANGED);

#if 0
    // 打印通道数和数据类型
    // ..\openCVDemo\modules\openCVManager\OpenCVManager.cpp 9166 "2024-10-31 20:07:42:619" 4 24 24
    LOG << matMask.channels() << matMask.type() << CV_8UC4; // 4 24
    // 打印mask蒙版行像素，隔一定行数打一次
    for(int row = 0; row < matMask.rows; row += 10)
    {
        for(int col = 100; col < matMask.cols; col++)
        {
            int r = matMask.at(row, col)[2];
            int g = matMask.at(row, col)[1];
            int b = matMask.at(row, col)[0];
            int a = matMask.at(row, col)[3];
            LOG << "row:" << row << ", col:" << col << "r(rgba):" << r << g << b << a;
            break;
        }
    }
#endif

    // 图片较大，缩为原来的0.5倍
    cv::resize(matLeft, matLeft, cv::Size(0, 0), 0.5, 0.5);
    cv::resize(matRight, matRight, cv::Size(0, 0), 0.5, 0.5);
    cv::resize(matMask1, matMask1, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask2, matMask2, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask3, matMask3, cv::Size(matLeft.cols, matLeft.rows));
    cv::resize(matMask4, matMask4, cv::Size(matLeft.cols, matLeft.rows));
    // 底图，扩大500横向，方便移动
    cv::Mat matResult = cv::Mat(matLeft.rows, matLeft.cols + 500, CV_8UC3);

    // 第一张图
    int key = 0;
    int x = 0;
    while(true)
    {
        // 副本，每次都要重新清空来调整
        cv::Mat matResult2 = matResult.clone();
#if 1
        // 第一张图，直接比例赋值，因为底图为0
        for(int row = 0; row < matLeft.rows; row++)
        {
            for(int col = 0; col < matLeft.cols; col++)
            {
                double r = matMask1.at(row, col)[2] / 255.0f;
//                double r = matMask2.at(row, col)[1] / 255.0f;
//                double r = matMask3.at(row, col)[0] / 255.0f;
//                double r = matMask4.at(row, col)[0] / 255.0f;
                matResult2.at(row, col)[0] = (matLeft.at(row, col)[0] * r);
                matResult2.at(row, col)[1] = (matLeft.at(row, col)[1] * r);
                matResult2.at(row, col)[2] = (uchar)(matLeft.at(row, col)[2] * r);
            }
        }
#endif
#if 0
        // 第二张图，加法，因为底图为原图了
        for(int row = 0; row < matRight.rows; row++)
        {
            for(int col = 0; col < matRight.cols; col++)
            {
                double g = matMask2.at(row, col)[1] / 255.0f;
                // 偏移了x坐标
                matResult2.at(row, col + x)[0] += matRight.at(row, col)[0] * g;
                matResult2.at(row, col + x)[1] += matRight.at(row, col)[1] * g;
                matResult2.at(row, col + x)[2] += matRight.at(row, col)[2] * g;
            }
        }
#endif
#if 1
        // 第二张图，加法，因为底图为原图了（优化）
        for(int row = 0; row < matRight.rows; row++)
        {
            for(int col = 0; col < matRight.cols; col++)
            {
                double r2;
                if(x + col <= matLeft.cols)
                {
                    r2 = (255 - matMask1.at(row, col + x)[2]) / 255.0f;
                }else{
                    r2 = 1.0f;
                }
                // 偏移了x坐标
                matResult2.at(row, col + x)[0] += matRight.at(row, col)[0] * r2;
                matResult2.at(row, col + x)[1] += matRight.at(row, col)[1] * r2;
                matResult2.at(row, col + x)[2] += matRight.at(row, col)[2] * r2;
            }
        }
#endif

//        cv::imshow("matMask1", matMask1);
//        cv::imshow("matLeft", matLeft);
        cv::imshow("matResult2", matResult2);
        key = cv::waitKey(0);
        if(key == 'a')
        {
            x--;
            if(x < 0)
            {
                x = 0;
            }
        }else if(key == 'd')
        {
            x++;
            if(x + matRight.cols > matResult2.cols)
            {
                x = matResult2.cols - matRight.cols;
            }
        }else if(key == 'q')
        {
            break;
        }
    }
}

```


 

# 工程模板v1\.72\.0




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213028-1642779473.png)



 

# 入坑




## 入坑一：读取通道rgba失败




### :[蓝猫机场加速器](https://dahelaoshi.com)问题：读取通道rgba失败




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213133-1465881826.png)




### 原因




  是uchar，转换成byte，而不是int  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213101-963297196.png)




## 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213022-1686106090.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213117-1663189476.png)




## 入坑二：读取通道一直是0，0，0，255




### 问题




  读取通道一直是0，0，0，255。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213074-1034434329.png)




### 原因




  弄了张图，还是255，然后发现是为了截图更清楚，弄了个边框，而我们打印正好是打印了0位置。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213224-797694494.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213170-953415970.png)




### 解决




  最终是要去掉边框，没边框就是空看不出，如下图：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213125-1962709530.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212896-881271429.png)




## 入坑三：过渡有黑线赋值不对




### 问题




  直接位置赋值，出现条纹  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213252-1985255800.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213259-1353518363.png)




### 原因




  类型是vec4b  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213026-1934307682.png)




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213129-1609028013.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213343-595979944.png)




## 入坑四：原图融合比例有黑线




### 问题




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213521-1396958581.png)




### 原因




  跟上面一样，mask蒙版是rgba的，需要vec4b  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213147-1045268460.png)




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125212939-84959988.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241114125213216-1824461400.png)



