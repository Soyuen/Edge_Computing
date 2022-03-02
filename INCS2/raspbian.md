# Intel神經運算棒第二代在樹梅派系統上進行推論
在這邊詳細紀錄Intel Neural Compute Stick 2(INC2)在樹梅派上進行推論和前置作業的步驟以及目前遇到各種問題的解決方式，各種踩坑彙整。
## 需要的配備
* 樹梅派 (Raspberry Pi)，試過第四代可行
* 神經運算棒第二代(Intel Neural Compute Stick 2)

## 軟體需求
* Python 3.5
* Cmake  3.7.2以上版本

## 安裝Python
官網中提到Python版本為3.5(沒試過3.5以上的)因此在這邊放上安裝3.5的步驟。  
**1.先安裝所需的工具**
```
$ sudo apt-get update
$ sudo apt-get install build-essential tk-dev
$ sudo apt-get install libncurses5-dev libncursesw5-dev libreadline6-dev
$ sudo apt-get install libdb5.3-dev libgdbm-dev libsqlite3-dev libssl-dev
$ sudo apt-get install libbz2-dev libexpat1-dev liblzma-dev zlib1g-dev
```
如果在安裝的時候出現錯誤，執行第一行更新後再繼續安裝下去。  
**2.下載Python3.5壓縮檔**  
可以進到[Python版本](https://www.python.org/ftp/python/)選擇，在這邊以3.5的最新版本3.5.10做示例。
```
$ wget https://www.python.org/ftp/python/3.5.10/Python-3.5.10.tgz
```
解壓縮檔案
```
$ tar zxvf Python-3.5.10.tgz
```
進入剛剛解壓縮好的檔案資料夾
```
$ cd Python-3.5.10
```
給定權限
```
$ sudo chmod -R 777 ./configure
```
配置安裝的路徑並安裝
```
$ ./configure --prefix=/usr/local/opt/python-3.5.2
$ make
$ sudo make install
```
## 安裝OpenVINO™ Toolkit
到[OpenVINO™ Toolkit packages storage](https://storage.openvinotoolkit.org/repositories/openvino/packages/)選擇想要下載的版本，下載的檔案選擇帶有"runtime_raspbian_p"名字的壓縮檔，如果懶得點進去選，可以在這裡點選[下載](https://storage.openvinotoolkit.org/repositories/openvino/packages/2021.4.2/l_openvino_toolkit_runtime_raspbian_p_2021.4.752.tgz)，此版本為2021.4.2(2021-11-12日更新)  

創建資料夾
```
$ sudo mkdir -p /opt/intel/openvino
```
剛剛下載的檔案通常會放在Downloads裡面，因此先進入Downloads資料夾(或者有指定放置的資料夾，就自己改)
```
$ cd ~/Downloads
```
解壓縮檔案到剛剛創建好的資料夾裡，這邊以前面提供的下載點為例子，如果是下載其他版本的話，把下面tgz檔改成自己下載的版本
```
$ sudo tar -xf  l_openvino_toolkit_runtime_raspbian_p_2021.4.752.tgz --strip 1 -C /opt/intel/openvino
```
## 下載其他所需軟體
下載Cmake
```
$ sudo apt install cmake
```
## 設置環境變量
這邊直接設置永久環境變量
```
$ echo "source /opt/intel/openvino/bin/setupvars.sh" >> ~/.bashrc
```
如果顯示以下圖片，代表環境設置成功  
![image](https://github.com/Soyuen/picture/blob/main/1.png)  

## 設置加速棒
將目前使用者新增到users群組中
```
$ sudo usermod -a -G users "$(whoami)"
```
要在加速棒上進行推理要先安裝腳本
```
$ sh /opt/intel/openvino/install_dependencies/install_NCS_udev_rules.sh
```

現在可以將加速棒插入樹莓派上了
## 建立與執行物件偵測範例程式
建立一個資料夾並進入該資料夾
```
$ sudo mkdir build && cd build
```
接下來重點來了，如果跟著官方網站給的教學，一定會錯誤，請照以下步驟執行

先給定openvino權限，後面改東西比較方便
```
$ sudo chmod -R 777 /opt/intel/openvino
```
若想要在**terminal**上變更檔案，建議使用vim編輯，首先先下載vim編輯器
```
$ sudo apt-get install vim
```
進入目的資料夾
```
cd /opt/intel/openvino_2021/deployment_tools/inference_engine/samples/cpp
```
我們要設定InferenceEngine的資料夾，否則如果照著官網做會出現以下錯誤  
![image](https://github.com/Soyuen/picture/blob/main/2.jpg)  


使用vim編輯器，打開CMakeLists.txt檔
```
vim CMakeLists.txt
```
會進入這樣子的一個畫面  
![image](https://github.com/Soyuen/picture/blob/main/3.jpg)  

按鍵盤**i**鍵進入編輯模式在project(Samples)下面打下這個程式碼
```
set(InferenceEngine_DIR /opt/intel/openvino/deployment_tools/inference_engine/share)
```
按**Esc**鍵並打:wq退出並儲存文件  

如果嫌以上步驟太麻煩因為是圖形化介面，直接從圖形化介面打開CmakeList.txt，就可以進入直接新增，因為剛剛把權限調整了，因此可以直接儲存檔案，不會有權限不足的問題。  
![image](https://github.com/Soyuen/picture/blob/main/4.jpg)  

完成之後就可以建立範例程式
```
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS="-march=armv7-a" /opt/intel/openvino/deployment_tools/inference_engine/samples/cpp
$ sudo make -j2 object_detection_sample_ssd
```
![image](https://github.com/Soyuen/picture/blob/main/5.jpg)  


接下來可以下載官方網站提供的openzoo(就是各種模型的集大成)，並下載所需套件以及臉部偵測模型，第二行中如果前面下載的toolkit不是2021.4.2版請改成自己的版本，requirements位置可能也有所改變。
```
$ cd /opt/intel/openvino
$ git clone --depth 1 -b 2021.4.2 https://github.com/openvinotoolkit/open_model_zoo
$ cd open_model_zoo/tools/downloader
$ python3 -m pip install -r requirements.in
$ python3 downloader.py --name face-detection-adas-0001
```
準備照片，這邊就不附照片了，照片位置不同的話，將照片改成自己存放的路徑  
執行推論

```
$ /opt/intel/openvino/deployment_tools/inference_engine/samples/cpp/armv7l/Release/object_detection_sample_ssd -m /opt/intel/openvino/open_model_zoo/tools/downloader/intel/face-detection-adas-0001/FP16/face-detection-adas-0001.xml -d MYRIAD -i /home/pi/Downloads/image.jpeg
```

這樣子就可以執行出openvino的範例程式了

