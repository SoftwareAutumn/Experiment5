# IHS遥感图像融合

## 一、任务描述

使用IHS变换，对下面图像进行融合。

*多光谱图像*

![American_MUL](/img5/American_MUL.bmp)      

*全色图像*              

![American_PAN](/img5/American_PAN.bmp)   

## 二、实验步骤                     

(1) 将多光谱影像重采样到与全色影像具有相同的分辨率；

(2) 将多光谱图像的Ｒ、Ｇ、Ｂ三个波段转换到IHS空间，得到Ｉ、Ｈ、Ｓ三个分量；

(3) 以Ｉ分量为参考，对全色影像进行直方图匹配

(4) 用全色影像替代Ｉ分量，然后同Ｈ、Ｓ分量一起逆变换到RGB空间，从而得到融合图像。   

RGB与IHS变换的基本公式：

![tran](/img5/tran.jpg)               

## 三、实验代码         

```
#include "pch.h"
#include <iostream>
#include <cmath>
#include "./gdal/gdal_priv.h"
#pragma comment(lib, "gdal_i.lib")
using namespace std;

int main()
{
	char* mulPath = (char*)"American_Mul.bmp";
	char* panPath = (char*)"American_Pan.bmp";
	char* fusPath = (char*)"American_Fus.jpg";

	GDALAllRegister();

	// basic parameters
	GDALDataset *poMulDS, *poPanDS, *poFusDS;
	int imgXlen, imgYlen;
	int i;
	float *bandR, *bandG, *bandB;
	float *bandH, *bandS;
	float *bandP;

	// open datasets
	poMulDS = (GDALDataset*)GDALOpenShared(mulPath, GA_ReadOnly);
	poPanDS = (GDALDataset*)GDALOpenShared(panPath, GA_ReadOnly);
	imgXlen = poMulDS->GetRasterXSize();
	imgYlen = poMulDS->GetRasterYSize();
	poFusDS = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(
		fusPath, imgXlen, imgYlen, 3, GDT_Byte, NULL);

	// allocating memory
	bandR = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));
	bandG = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));
	bandB = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));
	bandP = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));
	bandH = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));
	bandS = (float*)CPLMalloc(imgXlen*imgYlen * sizeof(float));

	poMulDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,
		bandR, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poMulDS->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,
		bandG, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poMulDS->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,
		bandB, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poPanDS->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, imgXlen, imgYlen,
		bandP, imgXlen, imgYlen, GDT_Float32, 0, 0);

	for (i = 0; i < imgXlen*imgYlen; i++)
	{
		bandH[i] = -sqrt(2.0f) / 6.0f*bandR[i] - sqrt(2.0f) / 6.0f*bandG[i] + sqrt(2.0f) / 3.0f*bandB[i];
		bandS[i] = 1.0f / sqrt(2.0f)*bandR[i] - 1 / sqrt(2.0f)*bandG[i];

		bandR[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] + 1.0f / sqrt(2.0f)*bandS[i];
		bandG[i] = bandP[i] - 1.0f / sqrt(2.0f)*bandH[i] - 1.0f / sqrt(2.0f)*bandS[i];
		bandB[i] = bandP[i] + sqrt(2.0f)*bandH[i];
	}

	poFusDS->GetRasterBand(1)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,
		bandR, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poFusDS->GetRasterBand(2)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,
		bandG, imgXlen, imgYlen, GDT_Float32, 0, 0);
	poFusDS->GetRasterBand(3)->RasterIO(GF_Write, 0, 0, imgXlen, imgYlen,
		bandB, imgXlen, imgYlen, GDT_Float32, 0, 0);

	CPLFree(bandR);
	CPLFree(bandG);
	CPLFree(bandB);
	CPLFree(bandH);
	CPLFree(bandS);
	CPLFree(bandP);

	GDALClose(poMulDS);
	GDALClose(poPanDS);
	GDALClose(poFusDS);


	return 0;
}
```

##   四、实验结果

![American_Fus](/img5/American_Fus.jpg)

