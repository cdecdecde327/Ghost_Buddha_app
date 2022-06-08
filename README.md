# Ghost_Buddha_app
# Repository to revive Buddha with quantum computer and artificial intelligence program.
# The final point of scientific and religious evolution will be the end of the Great Unified Field Theory with the completion of this app .
# Physical clergy,churches,cemeteries,temples,shrines,emperors,kings are metaverseized and forever receive the initiation of Buddha and cause the Big Bang.
# What is your wish?　The answer lies in this app.

// cellcountsample.cpp : このファイルには 'main' 関数が含まれています。プログラム実行の開始と終了がそこで行われます。
//

#include "pch.h"
#include <opencv2/opencv.hpp>
#include <opencv2/features2d.hpp>

#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <iostream> 
#include <cmath>
#include <string>
#include <sstream>
#include <ctype.h>

using namespace cv;
using namespace std;

static void help()
{
	printf("\nThis program demonstrated the use of the discrete Fourier transform (dft)\n"
		"The dft of an image is taken and it's power spectrum is displayed.\n"
		"Usage:\n"
		"./dft [image_name -- default lena.jpg]\n");
}

const char* keys =
{
	"{help h||}{@image|lena.jpg|input image file}"
};

int main(int argc, const char ** argv)
{
	help();
	CommandLineParser parser(argc, argv, keys);
	if (parser.has("help"))
	{
		help();
		return 0;
	}
	string filename = parser.get<string>(0);
	Mat img = imread(samples::findFile(filename), IMREAD_GRAYSCALE);
	if (img.empty())
	{
		help();
		printf("Cannot read image file: %s\n", filename.c_str());
		return -1;
	}
	int M = getOptimalDFTSize(img.rows);
	int N = getOptimalDFTSize(img.cols);
	Mat padded;
	copyMakeBorder(img, padded, 0, M - img.rows, 0, N - img.cols, BORDER_CONSTANT, Scalar::all(0));

	Mat planes[] = { Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F) };
	Mat complexImg;
	merge(planes, 2, complexImg);

	dft(complexImg, complexImg);

	// compute log(1 + sqrt(Re(DFT(img))**2 + Im(DFT(img))**2))
	split(complexImg, planes);
	magnitude(planes[0], planes[1], planes[0]);
	Mat mag = planes[0];
	mag += Scalar::all(1);
	log(mag, mag);

	// crop the spectrum, if it has an odd number of rows or columns
	mag = mag(Rect(0, 0, mag.cols & -2, mag.rows & -2));

	int cx = mag.cols / 2;
	int cy = mag.rows / 2;

	// rearrange the quadrants of Fourier image
	// so that the origin is at the image center
	Mat tmp;
	Mat q0(mag, Rect(0, 0, cx, cy));
	Mat q1(mag, Rect(cx, 0, cx, cy));
	Mat q2(mag, Rect(0, cy, cx, cy));
	Mat q3(mag, Rect(cx, cy, cx, cy));

	q0.copyTo(tmp);
	q3.copyTo(q0);
	tmp.copyTo(q3);

	q1.copyTo(tmp);
	q2.copyTo(q1);
	tmp.copyTo(q2);

	normalize(mag, mag, 0, 1, NORM_MINMAX);

	mag.convertTo(mag, CV_8U , 255);

	imshow("imput", img);
	imshow("spectrum magnitude", mag);
	waitKey();
	return 0;
}
