free=fire
xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Filter Include="Source Files">
      <UniqueIdentifier>{4FC737F1-C7A5-4376-A066-2A32D752A2FF}</UniqueIdentifier>
      <Extensions>cpp;c;cc;cxx;def;odl;idl;hpj;bat;asm;asmx</Extensions>
    </Filter>
    <Filter Include="Header Files">
      <UniqueIdentifier>{93995380-89BD-4b04-88EB-625FBE52EBFB}</UniqueIdentifier>
      <Extensions>h;hh;hpp;hxx;hm;inl;inc;xsd</Extensions>
    </Filter>
    <Filter Include="Resource Files">
      <UniqueIdentifier>{67DA6AB6-F800-4c08-8B7A-83BB121AAD01}</UniqueIdentifier>
      <Extensions>rc;ico;cur;bmp;dlg;rc2;rct;bin;rgs;gif;jpg;jpeg;jpe;resx;tiff;tif;png;wav</Extensions>
    </Filter>
  </ItemGroup> 
  <ItemGroup>
    <ClInclude Include="utils.h">
      <Filter>Header Files</Filter>
    </ClInclude>
    <ClInclude Include="interception.h">
      <Filter>Header Files</Filter>
    </ClInclude>
    <ClInclude Include="accel.h">
      <Filter>Header Files</Filter>
    </ClInclude>
  </ItemGroup>
  <ItemGroup>
    <ClCompile Include="utils.cpp">
      <Filter>Source Files</Filter>
    </ClCompile>
    <ClCompile Include="accel.cpp">
      <Filter>Source Files</Filter>
    </ClCompile>
  </ItemGroup>
</Project>{4FC737F1-C7A5-4376-A066-2A32D752A2FF} cpp;c;cc;cxx;def;odl;idl;hpj;bat;asm;asmx {93995380-89BD-4b04-88EB-625FBE52EBFB} h;hh;hpp;hxx;hm;inl;inc;xsd {67DA6AB6-F800-4c08-8B7A-83BB121AAD01} rc;ico;cur;bmp;dlg;rc2;rct;bin;rgs;gif;jpg;jpeg;jpe;resx;tiff;tif;png;wav Arquivos de Cabeçalho Arquivos de Cabeçalho Arquivos de Cabeçalho Arquivos de Origem Arquivos de Origem free=fire.alto=headshot> águadodeserto>
from PySide2 import QtWidgets, QtCore, QtGui
from time import sleep
from setup import query_

config = query_()

class Mira(QtWidgets.QDialog):
    def __init__(self, parente=None):
        self.parente = parente
        super(Mira, self).__init__(parent=self.parente)
        self.setWindowFlags(
            QtCore.Qt.WindowTransparentForInput 
            | QtCore.Qt.FramelessWindowHint 
            | QtCore.Qt.WindowStaysOnTopHint
        )
        self.resize(300, 600)
        self.setFocusPolicy(QtCore.Qt.ClickFocus)
        self.setAttribute(QtCore.Qt.WA_TranslucentBackground, True)
        self.lb = QtWidgets.QPushButton(self)
        self.lb.setIcon(QtGui.QPixmap(config["mira"]))
        self.lb.setIconSize(QtCore.QSize(config['tamanho'], config['tamanho']))
        self.lb.setStyleSheet('background: rgba(0, 0, 0, 0);')
        self.lb.resize(60, 60)
        self.lb.move(config['posicao'][0], config['posicao'][1])
        self.show()self.parenteQtCore.Qt.WindowTransparentForInputQtCore.Qt.FramelessWindowHintQtCore.Qt.WindowStaysOnTopHintself.lbself.show.freefire
include "interception.h"
#include "utils.h"
#include <windows.h>
#include <math.h>
#include <iostream>

int main()
{
	InterceptionContext context;
	InterceptionDevice device;
	InterceptionStroke stroke;

	raise_process_priority();

	context = interception_create_context();

	// interception_set_filter(context, interception_is_keyboard, INTERCEPTION_FILTER_KEY_DOWN | INTERCEPTION_FILTER_KEY_UP);
	interception_set_filter(context, interception_is_mouse, INTERCEPTION_FILTER_MOUSE_MOVE);


	int
		var_accelMode = 0;

	double
		frameTime_ms = 0,
		dx,
		dy,
		accelSens,
		rate,
		power,
		a,					//Taunty called it 'a' and I'm not creative
		b,					//var_accel/abs(a)
		carryX = 0,
		carryY = 0,
		reducedX = 0,
		reducedY = 0,
		var_sens = 1,
		var_accel = 0,
		var_senscap = 0,
		var_offset = 0,
		var_power = 2,
		var_preScaleX = 1,
		var_preScaleY = 1,
		var_postScaleX = 1,
		var_postScaleY = 1,
		var_angle = 0,
		var_angleSnap = 0,
		var_speedCap = 0,
		pi = 3.141592653589793238462643383279502884197169399375105820974944592307816406,
		hypot,
		angle,
		newangle,
		variableValue;

	bool debugOutput = 0, garbageFile = 0;
	char variableName[24];
	COORD coord;

	HANDLE hConsole;
	hConsole = GetStdHandle(STD_OUTPUT_HANDLE);

	
	CONSOLE_FONT_INFOEX cfi;
	cfi.cbSize = sizeof cfi;
	cfi.nFont = 0;
	cfi.dwFontSize.X = 0;
	cfi.dwFontSize.Y = 14;
	cfi.FontFamily = FF_DONTCARE;
	cfi.FontWeight = FW_NORMAL;
	wcscpy(cfi.FaceName, L"Consolas");
	SetCurrentConsoleFontEx(hConsole, FALSE, &cfi);
	

	coord.X = 80;
	coord.Y = 25;
	SetConsoleScreenBufferSize(hConsole, coord);
	
	SMALL_RECT conSize;

	conSize.Left = 0;
	conSize.Top = 0;
	conSize.Right = coord.X - 1;
	conSize.Bottom = coord.Y - 1;

	SetConsoleWindowInfo(hConsole, TRUE, &conSize);

	SetConsoleTextAttribute(hConsole, 0x0f);
	printf("povohat's interception mouse accel\nAdditional contributions by Sidiouth & _m00se_\n==============================================\n\n");
	SetConsoleTextAttribute(hConsole, 0x08);


	printf("Opening settings file...\n");


	// read variables once at runtime
	FILE *fp;

	if ((fp = fopen("settings.txt", "r+")) == NULL) {
		SetConsoleTextAttribute(hConsole, 0x04);
		printf("* Cannot read from settings file. Using defaults.\n");
		SetConsoleTextAttribute(hConsole, 0x08);
	}
	else
	{
		for (int i = 0; i < 99 && fscanf(fp, "%s = %lf", &variableName, &variableValue) != EOF; i++) {		//Doesn't complain if a line in settings.txt is missing

			if (strcmp(variableName, "AccelMode") == 0)
			{
				var_accelMode = variableValue;
			}
			else if (strcmp(variableName, "Sensitivity") == 0)
			{
				var_sens = variableValue;
			}
			else if (strcmp(variableName, "Acceleration") == 0)
			{
				var_accel = variableValue;
			}
			else if (strcmp(variableName, "SensitivityCap") == 0)
			{
				var_senscap = variableValue;
			}
			else if (strcmp(variableName, "Offset") == 0)
			{
				var_offset = variableValue;
			}
			else if (strcmp(variableName, "Power") == 0)
			{
				var_power = variableValue;
			}
			else if (strcmp(variableName, "Pre-ScaleX") == 0)
			{
				var_preScaleX = variableValue;
			}
			else if (strcmp(variableName, "Pre-ScaleY") == 0)
			{
				var_preScaleY = variableValue;
			}
			else if (strcmp(variableName, "Post-ScaleX") == 0)
			{
				var_postScaleX = variableValue;
			}
			else if (strcmp(variableName, "Post-ScaleY") == 0)
			{
				var_postScaleY = variableValue;
			}
			else if (strcmp(variableName, "AngleAdjustment") == 0)
			{
				var_angle = variableValue;
			}
			else if (strcmp(variableName, "AngleSnapping") == 0)
			{
				var_angleSnap = variableValue;
			}
			else if (strcmp(variableName, "SpeedCap") == 0)
			{
				var_speedCap = variableValue;
			}
			else if (strcmp(variableName, "FancyOutput") == 0)
			{
				if (variableValue != 0) {
					debugOutput = 1;
				}
				
			}
			else
			{
				garbageFile = 1;
			}
		}

		fclose(fp);

	}

	if (garbageFile) {
		SetConsoleTextAttribute(hConsole, 0x04);
		printf("* Your settings.txt has garbage in it which is being ignored\n");
		SetConsoleTextAttribute(hConsole, 0x08);
	}

	printf("\nYour settings are:\n");

	SetConsoleTextAttribute(hConsole, 0x02);
	printf("AccelMode: %i\nSensitivity: %f\nAcceleration: %f\nSensitivity Cap: %f\nOffset: %f\nPower: %f\nPre-Scale: x:%f, y:%f\nPost-Scale: x:%f, y:%f\nAngle Correction: %f\nAngle Snapping: %f\nSpeed Cap: %f\n\n", var_accelMode, var_sens, var_accel, var_senscap, var_offset, var_power, var_preScaleX, var_preScaleY, var_postScaleX, var_postScaleY, var_angle, var_angleSnap, var_speedCap);
	SetConsoleTextAttribute(hConsole, 0x08);


	SetConsoleTextAttribute(hConsole, 0x4f);
	printf(" [CTRL+C] to QUIT ");
	SetConsoleTextAttribute(hConsole, 0x08);

	if (!debugOutput) {
		printf("\n\nSet 'FancyOutput = 1' in settings.txt for realtime data\n(debug use only: may result in some latency)");
	}
	

	LARGE_INTEGER frameTime, oldFrameTime, PCfreq;

	QueryPerformanceCounter(&oldFrameTime);
	QueryPerformanceFrequency(&PCfreq);
	
	//Pre-loop calculations
	a = var_senscap - var_sens;
	b = var_accel / abs(a);
	power = var_power - 1 < 0 ? 0 : var_power - 1;

	while (interception_receive(context, device = interception_wait(context), &stroke, 1) > 0)
	{

		if (interception_is_mouse(device))
		{
			InterceptionMouseStroke &mstroke = *(InterceptionMouseStroke *)&stroke;

			if (!(mstroke.flags & INTERCEPTION_MOUSE_MOVE_ABSOLUTE)) {

				// figure out frametime
				QueryPerformanceCounter(&frameTime);
				frameTime_ms = (double) (frameTime.QuadPart - oldFrameTime.QuadPart) * 1000.0 / PCfreq.QuadPart;
				if (frameTime_ms > 200)
					frameTime_ms = 200;

				// retrieve new mouse data
				dx = (double) mstroke.x;
				dy = (double) mstroke.y;

				// angle correction
				if (var_angle) {
					hypot = sqrt(dx*dx + dy*dy); // convert to polar
					angle = atan2(dy, dx);

					angle += (var_angle * pi / 180); // apply adjustment in radians

					dx = hypot * cos(angle); // convert back to cartesian
					dy = hypot * sin(angle);
				}

				// angle snapping
				if (var_angleSnap) {
					hypot = sqrt(dx*dx + dy*dy); // convert to polar
					newangle = angle = atan2(dy, dx);


					if (fabs(cos(angle)) < (var_angleSnap*pi / 180)) {	// test for vertical
						if (sin(angle) > 0) {
							newangle = pi / 2;
						}
						else {
							newangle = 3 * pi / 2;
						}
					}
					else
						if (fabs(sin(angle)) < (var_angleSnap*pi / 180)) {	// test for horizontal
							if (cos(angle) < 0) {
								newangle = pi;
							}
							else {
								newangle = 0;
							}
						}

					dx = hypot * cos(newangle); // convert back to cartesian
					dy = hypot * sin(newangle);

					if (debugOutput) {

						coord.X = 40;
						coord.Y = 14;
						SetConsoleCursorPosition(hConsole, coord);
						if (angle - newangle != 0) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Snapped");
						}
						else {
							printf("       ");
						}
						SetConsoleTextAttribute(hConsole, 0x08);

					}
						

				}

				// apply pre-scale
				dx *= var_preScaleX;
				dy *= var_preScaleY;

				// apply speedcap
				if (var_speedCap) {
					rate = sqrt(dx*dx + dy*dy);

					if (debugOutput) {
						coord.X = 40;
						coord.Y = 15;
						SetConsoleCursorPosition(hConsole, coord);
					}

					if (rate >= var_speedCap) {
						dx *= var_speedCap / rate;
						dy *= var_speedCap / rate;
						if (debugOutput) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Capped");
							SetConsoleTextAttribute(hConsole, 0x08);
						}
					}
					else {
						if (debugOutput) {
							printf("      ");
						}							
					}
				}

				// apply accel
				accelSens = var_sens;							// start with in-game sens so accel calc scales the same
				if (var_accel > 0) {
					rate = sqrt(dx*dx + dy*dy) / frameTime_ms;	// calculate velocity of mouse based on deltas
					rate -= var_offset;							// offset affects the rate that accel sees
					if (rate > 0) {
						switch (var_accelMode) {
						case 0:									//Original InterAccel acceleration
							accelSens += pow((rate*var_accel), power);
							break;
						case 1:									//TauntyArmordillo's natural acceleration
							accelSens += a - (a * exp((-rate*b)));
							break;
						case 2:									//Natural Log acceleration
							accelSens += log((rate*var_accel) + 1);
							break;
						}
					}

					if (debugOutput) {
						coord.X = 40;
						coord.Y = 8;
						SetConsoleCursorPosition(hConsole, coord);
					}

					if (var_senscap > 0 && accelSens >= var_senscap) {
						accelSens = var_senscap;				// clamp post-accel sensitivity at senscap
						if (debugOutput) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Capped");
						}
					}
					else {
						if (debugOutput) {
							printf("      ");
						}
					}

					if (debugOutput) {
						SetConsoleTextAttribute(hConsole, 0x08);
					}


				}
				accelSens /= var_sens;							// divide by in-game sens as game will multiply it out
				dx *= accelSens;								// apply accel to horizontal
				dy *= accelSens;

				// apply post-scale
				dx *= var_postScaleX;
				dy *= var_postScaleY;

				// add remainder from previous cycle
				dx += carryX;
				dy += carryY;

				// reduce movement to whole number
				reducedX = round(dx);
				reducedY = round(dy);

				// remainder gets passed into next cycle
				carryX = dx - reducedX;
				carryY = dy - reducedY;

				if (debugOutput) {
					coord.X = 0;
					coord.Y = 20;
					SetConsoleCursorPosition(hConsole, coord);
					SetConsoleTextAttribute(hConsole, 0x08);
					printf("input    - X: %05d   Y: %05d\n", mstroke.x, mstroke.y);
					printf("output   - X: %05d   Y: %05d    accel sens: %.3f      \n", (int)reducedX, (int)reducedY, accelSens);
					printf("subpixel - X: %.3f   Y: %.3f    frame time: %.3f      ", carryX, carryY, frameTime_ms);
					SetConsoleTextAttribute(hConsole, 0x08);


					coord.X = 40;
					coord.Y = 7;
					SetConsoleCursorPosition(hConsole, coord);
					if (accelSens > 1) {
						SetConsoleTextAttribute(hConsole, 0x2f);
						printf("Accel +");
					}
					else if (accelSens < 1) {
						SetConsoleTextAttribute(hConsole, 0x4f);
						printf("Accel -");
					}
					else {
						printf("       ");
					}
					SetConsoleTextAttribute(hConsole, 0x08);

				}

				// output new counts
				mstroke.x = (int)reducedX;
				mstroke.y = (int)reducedY;

				oldFrameTime = frameTime;
			}

			interception_send(context, device, &stroke, 1);
		} 
	}

	interception_destroy_context(context);

	return 0;
}include "interception.h"
#include "utils.h"
#include <windows.h>
#include <math.h>
#include <iostream>

int main()
{
	InterceptionContext context;
	InterceptionDevice device;
	InterceptionStroke stroke;

	raise_process_priority();

	context = interception_create_context();

	// interception_set_filter(context, interception_is_keyboard, INTERCEPTION_FILTER_KEY_DOWN | INTERCEPTION_FILTER_KEY_UP);
	interception_set_filter(context, interception_is_mouse, INTERCEPTION_FILTER_MOUSE_MOVE);


	int
		var_accelMode = 0;

	double
		frameTime_ms = 0,
		dx,
		dy,
		accelSens,
		rate,
		power,
		a,					//Taunty called it 'a' and I'm not creative
		b,					//var_accel/abs(a)
		carryX = 0,
		carryY = 0,
		reducedX = 0,
		reducedY = 0,
		var_sens = 1,
		var_accel = 0,
		var_senscap = 0,
		var_offset = 0,
		var_power = 2,
		var_preScaleX = 1,
		var_preScaleY = 1,
		var_postScaleX = 1,
		var_postScaleY = 1,
		var_angle = 0,
		var_angleSnap = 0,
		var_speedCap = 0,
		pi = 3.141592653589793238462643383279502884197169399375105820974944592307816406,
		hypot,
		angle,
		newangle,
		variableValue;

	bool debugOutput = 0, garbageFile = 0;
	char variableName[24];
	COORD coord;

	HANDLE hConsole;
	hConsole = GetStdHandle(STD_OUTPUT_HANDLE);

	
	CONSOLE_FONT_INFOEX cfi;
	cfi.cbSize = sizeof cfi;
	cfi.nFont = 0;
	cfi.dwFontSize.X = 0;
	cfi.dwFontSize.Y = 14;
	cfi.FontFamily = FF_DONTCARE;
	cfi.FontWeight = FW_NORMAL;
	wcscpy(cfi.FaceName, L"Consolas");
	SetCurrentConsoleFontEx(hConsole, FALSE, &cfi);
	

	coord.X = 80;
	coord.Y = 25;
	SetConsoleScreenBufferSize(hConsole, coord);
	
	SMALL_RECT conSize;

	conSize.Left = 0;
	conSize.Top = 0;
	conSize.Right = coord.X - 1;
	conSize.Bottom = coord.Y - 1;

	SetConsoleWindowInfo(hConsole, TRUE, &conSize);

	SetConsoleTextAttribute(hConsole, 0x0f);
	printf("povohat's interception mouse accel\nAdditional contributions by Sidiouth & _m00se_\n==============================================\n\n");
	SetConsoleTextAttribute(hConsole, 0x08);


	printf("Opening settings file...\n");


	// read variables once at runtime
	FILE *fp;

	if ((fp = fopen("settings.txt", "r+")) == NULL) {
		SetConsoleTextAttribute(hConsole, 0x04);
		printf("* Cannot read from settings file. Using defaults.\n");
		SetConsoleTextAttribute(hConsole, 0x08);
	}
	else
	{
		for (int i = 0; i < 99 && fscanf(fp, "%s = %lf", &variableName, &variableValue) != EOF; i++) {		//Doesn't complain if a line in settings.txt is missing

			if (strcmp(variableName, "AccelMode") == 0)
			{
				var_accelMode = variableValue;
			}
			else if (strcmp(variableName, "Sensitivity") == 0)
			{
				var_sens = variableValue;
			}
			else if (strcmp(variableName, "Acceleration") == 0)
			{
				var_accel = variableValue;
			}
			else if (strcmp(variableName, "SensitivityCap") == 0)
			{
				var_senscap = variableValue;
			}
			else if (strcmp(variableName, "Offset") == 0)
			{
				var_offset = variableValue;
			}
			else if (strcmp(variableName, "Power") == 0)
			{
				var_power = variableValue;
			}
			else if (strcmp(variableName, "Pre-ScaleX") == 0)
			{
				var_preScaleX = variableValue;
			}
			else if (strcmp(variableName, "Pre-ScaleY") == 0)
			{
				var_preScaleY = variableValue;
			}
			else if (strcmp(variableName, "Post-ScaleX") == 0)
			{
				var_postScaleX = variableValue;
			}
			else if (strcmp(variableName, "Post-ScaleY") == 0)
			{
				var_postScaleY = variableValue;
			}
			else if (strcmp(variableName, "AngleAdjustment") == 0)
			{
				var_angle = variableValue;
			}
			else if (strcmp(variableName, "AngleSnapping") == 0)
			{
				var_angleSnap = variableValue;
			}
			else if (strcmp(variableName, "SpeedCap") == 0)
			{
				var_speedCap = variableValue;
			}
			else if (strcmp(variableName, "FancyOutput") == 0)
			{
				if (variableValue != 0) {
					debugOutput = 1;
				}
				
			}
			else
			{
				garbageFile = 1;
			}
		}

		fclose(fp);

	}

	if (garbageFile) {
		SetConsoleTextAttribute(hConsole, 0x04);
		printf("* Your settings.txt has garbage in it which is being ignored\n");
		SetConsoleTextAttribute(hConsole, 0x08);
	}

	printf("\nYour settings are:\n");

	SetConsoleTextAttribute(hConsole, 0x02);
	printf("AccelMode: %i\nSensitivity: %f\nAcceleration: %f\nSensitivity Cap: %f\nOffset: %f\nPower: %f\nPre-Scale: x:%f, y:%f\nPost-Scale: x:%f, y:%f\nAngle Correction: %f\nAngle Snapping: %f\nSpeed Cap: %f\n\n", var_accelMode, var_sens, var_accel, var_senscap, var_offset, var_power, var_preScaleX, var_preScaleY, var_postScaleX, var_postScaleY, var_angle, var_angleSnap, var_speedCap);
	SetConsoleTextAttribute(hConsole, 0x08);


	SetConsoleTextAttribute(hConsole, 0x4f);
	printf(" [CTRL+C] to QUIT ");
	SetConsoleTextAttribute(hConsole, 0x08);

	if (!debugOutput) {
		printf("\n\nSet 'FancyOutput = 1' in settings.txt for realtime data\n(debug use only: may result in some latency)");
	}
	

	LARGE_INTEGER frameTime, oldFrameTime, PCfreq;

	QueryPerformanceCounter(&oldFrameTime);
	QueryPerformanceFrequency(&PCfreq);
	
	//Pre-loop calculations
	a = var_senscap - var_sens;
	b = var_accel / abs(a);
	power = var_power - 1 < 0 ? 0 : var_power - 1;

	while (interception_receive(context, device = interception_wait(context), &stroke, 1) > 0)
	{

		if (interception_is_mouse(device))
		{
			InterceptionMouseStroke &mstroke = *(InterceptionMouseStroke *)&stroke;

			if (!(mstroke.flags & INTERCEPTION_MOUSE_MOVE_ABSOLUTE)) {

				// figure out frametime
				QueryPerformanceCounter(&frameTime);
				frameTime_ms = (double) (frameTime.QuadPart - oldFrameTime.QuadPart) * 1000.0 / PCfreq.QuadPart;
				if (frameTime_ms > 200)
					frameTime_ms = 200;

				// retrieve new mouse data
				dx = (double) mstroke.x;
				dy = (double) mstroke.y;

				// angle correction
				if (var_angle) {
					hypot = sqrt(dx*dx + dy*dy); // convert to polar
					angle = atan2(dy, dx);

					angle += (var_angle * pi / 180); // apply adjustment in radians

					dx = hypot * cos(angle); // convert back to cartesian
					dy = hypot * sin(angle);
				}

				// angle snapping
				if (var_angleSnap) {
					hypot = sqrt(dx*dx + dy*dy); // convert to polar
					newangle = angle = atan2(dy, dx);


					if (fabs(cos(angle)) < (var_angleSnap*pi / 180)) {	// test for vertical
						if (sin(angle) > 0) {
							newangle = pi / 2;
						}
						else {
							newangle = 3 * pi / 2;
						}
					}
					else
						if (fabs(sin(angle)) < (var_angleSnap*pi / 180)) {	// test for horizontal
							if (cos(angle) < 0) {
								newangle = pi;
							}
							else {
								newangle = 0;
							}
						}

					dx = hypot * cos(newangle); // convert back to cartesian
					dy = hypot * sin(newangle);

					if (debugOutput) {

						coord.X = 40;
						coord.Y = 14;
						SetConsoleCursorPosition(hConsole, coord);
						if (angle - newangle != 0) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Snapped");
						}
						else {
							printf("       ");
						}
						SetConsoleTextAttribute(hConsole, 0x08);

					}
						

				}

				// apply pre-scale
				dx *= var_preScaleX;
				dy *= var_preScaleY;

				// apply speedcap
				if (var_speedCap) {
					rate = sqrt(dx*dx + dy*dy);

					if (debugOutput) {
						coord.X = 40;
						coord.Y = 15;
						SetConsoleCursorPosition(hConsole, coord);
					}

					if (rate >= var_speedCap) {
						dx *= var_speedCap / rate;
						dy *= var_speedCap / rate;
						if (debugOutput) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Capped");
							SetConsoleTextAttribute(hConsole, 0x08);
						}
					}
					else {
						if (debugOutput) {
							printf("      ");
						}							
					}
				}

				// apply accel
				accelSens = var_sens;							// start with in-game sens so accel calc scales the same
				if (var_accel > 0) {
					rate = sqrt(dx*dx + dy*dy) / frameTime_ms;	// calculate velocity of mouse based on deltas
					rate -= var_offset;							// offset affects the rate that accel sees
					if (rate > 0) {
						switch (var_accelMode) {
						case 0:									//Original InterAccel acceleration
							accelSens += pow((rate*var_accel), power);
							break;
						case 1:									//TauntyArmordillo's natural acceleration
							accelSens += a - (a * exp((-rate*b)));
							break;
						case 2:									//Natural Log acceleration
							accelSens += log((rate*var_accel) + 1);
							break;
						}
					}

					if (debugOutput) {
						coord.X = 40;
						coord.Y = 8;
						SetConsoleCursorPosition(hConsole, coord);
					}

					if (var_senscap > 0 && accelSens >= var_senscap) {
						accelSens = var_senscap;				// clamp post-accel sensitivity at senscap
						if (debugOutput) {
							SetConsoleTextAttribute(hConsole, 0x2f);
							printf("Capped");
						}
					}
					else {
						if (debugOutput) {
							printf("      ");
						}
					}

					if (debugOutput) {
						SetConsoleTextAttribute(hConsole, 0x08);
					}


				}
				accelSens /= var_sens;							// divide by in-game sens as game will multiply it out
				dx *= accelSens;								// apply accel to horizontal
				dy *= accelSens;

				// apply post-scale
				dx *= var_postScaleX;
				dy *= var_postScaleY;

				// add remainder from previous cycle
				dx += carryX;
				dy += carryY;

				// reduce movement to whole number
				reducedX = round(dx);
				reducedY = round(dy);

				// remainder gets passed into next cycle
				carryX = dx - reducedX;
				carryY = dy - reducedY;

				if (debugOutput) {
					coord.X = 0;
					coord.Y = 20;
					SetConsoleCursorPosition(hConsole, coord);
					SetConsoleTextAttribute(hConsole, 0x08);
					printf("input    - X: %05d   Y: %05d\n", mstroke.x, mstroke.y);
					printf("output   - X: %05d   Y: %05d    accel sens: %.3f      \n", (int)reducedX, (int)reducedY, accelSens);
					printf("subpixel - X: %.3f   Y: %.3f    frame time: %.3f      ", carryX, carryY, frameTime_ms);
					SetConsoleTextAttribute(hConsole, 0x08);


					coord.X = 40;
					coord.Y = 7;
					SetConsoleCursorPosition(hConsole, coord);
					if (accelSens > 1) {
						SetConsoleTextAttribute(hConsole, 0x2f);
						printf("Accel +");
					}
					else if (accelSens < 1) {
						SetConsoleTextAttribute(hConsole, 0x4f);
						printf("Accel -");
					}
					else {
						printf("       ");
					}
					SetConsoleTextAttribute(hConsole, 0x08);

				}

				// output new counts
				mstroke.x = (int)reducedX;
				mstroke.y = (int)reducedY;

				oldFrameTime = frameTime;
			}

			interception_send(context, device, &stroke, 1);
		} 
	}

	interception_destroy_context(context);

	return 0;
}dpi1400=1200
free fire
Taxa de tira na cabeça 100%
xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Filter Include="Source Files">
      <UniqueIdentifier>{4FC737F1-C7A5-4376-A066-2A32D752A2FF}</UniqueIdentifier>
      <Extensions>cpp;c;cc;cxx;def;odl;idl;hpj;bat;asm;asmx</Extensions>
    </Filter>
    <Filter Include="Header Files">
      <UniqueIdentifier>{93995380-89BD-4b04-88EB-625FBE52EBFB}</UniqueIdentifier>
      <Extensions>h;hh;hpp;hxx;hm;inl;inc;xsd</Extensions>
    </Filter>
    <Filter Include="Resource Files">
      <UniqueIdentifier>{67DA6AB6-F800-4c08-8B7A-83BB121AAD01}</UniqueIdentifier>
      <Extensions>rc;ico;cur;bmp;dlg;rc2;rct;bin;rgs;gif;jpg;jpeg;jpe;resx;tiff;tif;png;wav</Extensions>
    </Filter>
  </ItemGroup> 
  <ItemGroup>
    <ClInclude Include="utils.h">
      <Filter>Header Files</Filter>
    </ClInclude>
    <ClInclude Include="interception.h">
      <Filter>Header Files</Filter>
    </ClInclude>
    <ClInclude Include="accel.h">
      <Filter>Header Files</Filter>
    </ClInclude>
  </ItemGroup>
  <ItemGroup>
    <ClCompile Include="utils.cpp">
      <Filter>Source Files</Filter>
    </ClCompile>
    <ClCompile Include="accel.cpp">
      <Filter>Source Files</Filter>
    </ClCompile>
  </ItemGroup>
</Project>http://schemas.microsoft.com/developer/msbuild/2003
UnUnityFS    5.x.x 2018.4.11f1       ©q   ?   [   C  Q  ©  @    ð CAB-0223d31e5f5e773be3e1996bf08f4225   Å  ©           2018.4.11f1 
      Ž    ÿÿ—Ú_FˆäZWÈ´-OBIr—:   ò          7  €ÿÿÿÿ     €    H €« €ÿÿÿÿ   €  1  €1  €ÿÿÿÿ   @   Þ  € €          Q  €j  €          Õ €   ÿÿÿÿ    €   1  €1  €ÿÿÿÿ    @    Þ  € €            y €j  €            Þ  €      	    €   . €$      
    €   ñ  €-   ÿÿÿÿ    €   1  €1  €ÿÿÿÿ    €    Þ  € €   
          €j  €ÿÿÿÿ    €    H €›  €ÿÿÿÿ    €   1  €1  €ÿÿÿÿ   @    Þ  € €           Q  €j  €           9   
 €            Þ  €C               Þ  €P               y €\               Þ  €          €   . €$          €   9   b               Þ  €C               Þ  €P               y €\               Þ  €          €   . €$          €   ¦ €n               H €…   ÿÿÿÿ     €   1  €1  €ÿÿÿÿ!   @    Þ  € €   "        Q  €j  €   #        Õ €—   ÿÿÿÿ$    €   1  €1  €ÿÿÿÿ%    À    Þ  € €   &         H €j  €ÿÿÿÿ'    €   1  €1  €ÿÿÿÿ(   @    Þ  € €   )        Q  €j  €   *        L  €¦      +    @    Þ  €Ã      ,         Þ  €Ø      -         ñ  €ä   ÿÿÿÿ.    €   1  €1  €ÿÿÿÿ/    €    Þ  € €   0          €j  €ÿÿÿÿ1    €    H €›  €ÿÿÿÿ2    €   1  €1  €ÿÿÿÿ3   @    Þ  € €   4        Q  €j  €   5        H €
 €ÿÿÿÿ6    €   1  €1  €ÿÿÿÿ7   @    Þ  € €   8        Q  €j  €   9      AssetBundle m_PreloadTable m_FileID m_PathID m_Container AssetInfo preloadIndex preloadSize asset m_MainAsset m_RuntimeCompatibility m_AssetBundleName m_Dependencies m_IsStreamedSceneAssetBundle m_ExplicitDataLayout m_PathFlags m_SceneHashes 1    ÿÿHk¤á]½jêŠÁ d0X‰È	          O €7  €ÿÿÿÿ     €    H €« €ÿÿÿÿ   €  1  €1  €ÿÿÿÿ   @   Þ  € €          Q  €j  €          H €ê €ÿÿÿÿ   €  1  €1  €ÿÿÿÿ   @   Þ  € €          Q  €j  €                         ¼       HðsÚDTÀ   @˜                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        config/buffbehaviordata        HðsÚDT   ,   assets/resources/config/buffbehaviordata.csv           HðsÚDT                          config/buffbehaviordata                           BuffBehaviorData'˜  id,behavior_type,behavior_type_explanation,Des,show_active_icon,buff_effect,show_list_active_icon,buff_effect_type,behavior_icon,"stack_type",keep_after_death,trigger_delay,duration,value1,Parameter1Desc,value2,Parameter2Desc,value3,Parameter3Desc,params,value4,Parameter4Desc,value5,Parameter5Desc,value6,Parameter6Desc,value7,Parameter7Desc,value8,Parameter8Desc,value9,Parameter9Desc
Behavior ID,unread,unread,,unread,,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread,unread
1,1,unread,,TRUE,,FALSE,,UI_Weapon_skill_Damage_Increase,1,,0,-1,0.1,,,,,,,,,,,,,,,,,,
2,2,unread,,FALSE,,FALSE,,,1,,0,10,50,,,,,,,,,,,,,,,,,,
3,3,unread,,FALSE,,FALSE,,,1,,0,15,15,,,,,,,,,,,,,,,,,,
4,3,unread,,FALSE,,FALSE,,,1,,0,5,75,,,,,,,,,,,,,,,,,,
601,3,unread,,FALSE,,FALSE,,,1,,0,10,30,,,,,,,,,,,,,,,,,,
602,9,unread,,FALSE,,FALSE,,,1,,0,10,15,,,,,,,,,,,,,,,,,,
603,3,unread,,FALSE,,FALSE,,,1,,0,10,30,,,,,,,,,,,,,,,,,,
604,9,unread,,FALSE,,FALSE,,,1,,0,10,0,,,,,,,,,,,,,,,,,,
605,3,unread,T_40_W_NURSE_G,TRUE,,TRUE,,ui_icon_skill_Djmale_Awakening,1,,0,10,30,,,,,,,,,,,,,,,,,,
606,9,unread,,FALSE,,FALSE,,,1,,0,10,0,,,,,,,,,,,,,,,,,,
5,3,unread,,FALSE,,FALSE,,,1,,0,5,50,,,,,,,,,,,,,,,,,,
6,4,unread,,FALSE,,FALSE,,,1,,0,0,,,,,,,,,,,,,,,,,,,
7,5,unread,,FALSE,,FALSE,,,1,,0,0,,,,,,,,,,,,,,,,,,,
8,6,unread,,FALSE,,FALSE,,,1,,0,0,,,,,,,,,,,,,,,,,,,
9,7,unread,,FALSE,,FALSE,,,1,,0,0,,,,,,,,,,,,,,,,,,,
10,8,unread,,FALSE,,TRUE,,UI_icon_ingame_attack_buff,3,,0,10,65,,,,,,,,,,,,,,,,,,
11,9,unread,,FALSE,,TRUE,,UI_icon_ingame_mobile_buff,3,,0,8,40,,,,,,,,,,,,,,,,,,
12,10,unread,,FALSE,,TRUE,,UI_icon_ingame_UnDead_buff,0,,0,-1,1,unread,10,unread,,,,,,,,,,,,,,,
13,11,unread,,FALSE,,TRUE,,UI_icon_ingame_UnlimitedBullet_buff,3,,0,12,0,,,,,,,,,,,,,,,,,,
14,9,unread,,FALSE,,FALSE,,,4,,0,1,5,,,,,,,,,,,,,,,,,,
15,9,unread,,FALSE,,FALSE,,,4,,0,1,7,,,,,,,,,,,,,,,,,,
16,9,unread,,FALSE,,FALSE,,,4,,0,1,9,,,,,,,,,,,,,,,,,,
17,9,unread,,FALSE,,FALSE,,,4,,0,1,11,,,,,,,,,,,,,,,,,,
18,9,unread,,FALSE,,FALSE,,,4,,0,1,13,,,,,,,,,,,,,,,,,,
19,9,unread,,FALSE,,FALSE,,,4,,0,1,15,,,,,,,,,,,,,,,,,,
20,12,unread,,FALSE,,FALSE,,,1,TRUE,0,0,,,,,,,,,,,,,,,,,,,
21,13,unread,,FALSE,,FALSE,,,1,TRUE,0,0,,,,,,,,,,,,,,,,,,,
22,14,unread,,FALSE,,FALSE,,,1,,0,3,24,,5,,1,,,,,,,,,,,,,,
23,15,unread,,FALSE,,FALSE,,,1,,0,5,1,unread,1,unread,,,,,,,,,,,,,,,
24,15,unread,,FALSE,,FALSE,,,1,,0,5,2,,1,,,,,,,,,,,,,,,,
25,15,unread,,FALSE,,FALSE,,,1,,0,5,3,,1,,,,,,,,,,,,,,,,
26,15,unread,,FALSE,,FALSE,,,1,,0,5,4,,1,,,,,,,,,,,,,,,,
27,15,unread,,FALSE,,FALSE,,,1,,0,5,5,,1,,,,,,,,,,,,,,,,
28,16,unread,,FALSE,,FALSE,,,1,,0,-1,4.5,unread,5,unread,200,unread,,,,,,,,,,,,,
29,16,unread,,FALSE,,FALSE,,,1,,0,-1,4,,5,,200,,,,,,,,,,,,,,
30,16,unread,,FALSE,,FALSE,,,1,,0,-1,3.5,,5,,200,,,,,,,,,,,,,,
31,16,unread,,FALSE,,FALSE,,,1,,0,-1,3,,5,,200,,,,,,,,,,,,,,
32,16,unread,,FALSE,,FALSE,,,1,,0,-1,2.5,,5,,200,,,,,,,,,,,,,,
33,16,unread,,FALSE,,FALSE,,,1,,0,-1,2,,5,,200,,,,,,,,,,,,,,
34,17,unread,,FALSE,,FALSE,,,1,,0,5,200,unread,200,EP,,,,,,,,,,,,,,,
35,18,unread,,FALSE,,FALSE,,,1,,0,0,,,,,,,201:120;202:24;203:24;204:24;205:120;206:6;207:6;208:30;209:10;210:300;601:2;602:2;1201:2;1202:2,,,,,,,,,,,,
36,19,unread,,FALSE,,FALSE,,,0,,0,2,25,unread,25,EP,-10,,400;400,,,,,,,,,,,,
37,20,unread,,FALSE,,FALSE,,,1,,0,-1,3,unread,3,unread,,,,,,,,,,,,,,,
38,21,unread,T_24_O_WSS_AK47_D,TRUE,,FALSE,2,Icon_WSS_Ak47_Dragon,0,,0,-1,5,unread,,,,,VFX_INGAME_WSS_AK47DRAGON,,,,,,,,,,,,
39,22,unread,,TRUE,,,,FF_UI_Ingame_Speed_Buff_Ddamage,3,,0,10,0.1,,,,,,,,,,,,,,,,,,
40,23,unread,,TRUE,,,,FF_UI_Ingame_Speed_Buff_Hurt,3,,0,5,90,unread,,,,,,,,,,,,,,,,,
41,25,unread,,TRUE,,,,FF_UI_Ingame_Speed_Buff_CD,3,,0,10,1,unread,-0.5,unread,903,unread,,,,,,,,,,,,,
42,24,unread,,TRUE,,,,FF_UI_Ingame_Speed_Buff_Hurt,3,,0,10,30,unread,,,,,,,,,,,,,,,,,
43,8,unread,,TRUE,,,,FF_UI_Ingame_Speed_Buff_Ddamage,3,,0,5,30,unread,,,,,,,,,,,,,,,,,
44,26,unread,T_25_O_WSS_SCAR_D,TRUE,,FALSE,2,Icon_WSS_SCAR_Shark,2,,0,-1,120,unread,,,,,,,,,,,,,,,,,
45,27,unread,T_26_O_WSS_MP40_D,TRUE,,FALSE,2,Icon_WSS_MP40_Cobra,1,,0,-1,1,unread,60,unread,,,,,,,,,,,,,,,
46,28,unread,,,,,,,4,,0,4,4,unread,,,,,,,,,,,,,,,,,
47,28,unread,,,,,,,4,,0,4,5,unread,,,,,,,,,,,,,,,,,
48,28,unread,,,,,,,4,,0,4,6,unread,,,,,,,,,,,,,,,,,
49,28,unread,,,,,,,4,,0,4,7,unread,,,,,,,,,,,,,,,,,
50,28,unread,,,,,,,4,,0,4,8,unread,,,,,,,,,,,,,,,,,
51,28,unread,,,,,,,4,,0,4,9,unread,,,,,,,,,,,,,,,,,
52,29,WWTaskForbid,,FALSE,,,,,1,,0,-1,,,,,,,,,,,,,,,,,,,
53,30,WWMapForbid,,FALSE,,,,,1,,0,30,,,,,,,,,,,,,,,,,,,
54,31,WWFog,,FALSE,,,,,1,,0,30,,,,,,,,,,,,,,,,,,,
55,21,unread,T_24_O_WSS_AK47_D,TRUE,,FALSE,2,Icon_WSS_Ak47_Dragon,0,,0,-1,8,unread,,,,,VFX_INGAME_WSS_AK47DRAGON,,,,,,,,,,,,
56,32,unread,,,,,,,0,,0,-1,,,,,,,,,,,,,,,,,,,
57,33,unread,,,,,,,0,,0,-1,,,,,,,,,,,,,,,,,,,
58,15,unread,,FALSE,,FALSE,,,1,,0,5,6,,1,,,,,,,,,,,,,,,,
59,34,unread,T_28_W_WSS_XM8_D,TRUE,,FALSE,2,Icon_WSS_XM8_Thunder,1,,0,-1,1,unread,2,unread,,,,,,,,,,,,,,,
60,35,unread,,FALSE,,FALSE,,,2,,0,-1,10,unread,,,,,,,,,,,,,,,,,
61,36,unread,,FALSE,,FALSE,,,2,,0,-1,10,unread,,,,,,,,,,,,,,,,,
62,37,unread,,FALSE,,FALSE,,,2,,0,-1,50,unread,,,,,,,,,,,,,,,,,
63,38,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
64,39,unread,,FALSE,,FALSE,,,2,,0,-1,100,unread,,,,,,,,,,,,,,,,,
65,40,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
66,41,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
67,42,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
68,43,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
69,44,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
70,45,unread,,FALSE,,FALSE,,,2,,0,0,150,unread,,,,,,,,,,,,,,,,,
71,46,unread,,FALSE,,FALSE,,,2,,0,0,100,unread,,,,,,,,,,,,,,,,,
72,47,unread,,FALSE,,FALSE,,,2,,0,-1,400,unread,,,,,,,,,,,,,,,,,
73,48,unread,,FALSE,,FALSE,,,2,,0,-1,,unread,,,,,,,,,,,,,,,,,
74,49,unread,,FALSE,,FALSE,,,2,,0,-1,200,unread,,,,,,,,,,,,,,,,,
75,50,unread,,FALSE,,FALSE,,,2,,0,-1,,unread,,,,,,,,,,,,,,,,,
76,51,unread,,FALSE,,FALSE,,,2,,0,-1,100,unread,,,,,,,,,,,,,,,,,
77,52,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
78,53,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
79,54,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,,,,,,,,,,,,,,,,,
80,55,unread,,FALSE,,FALSE,,,2,,0,0,25,unread,,,,,,,,,,,,,,,,,
81,55,unread,,FALSE,,FALSE,,,2,,0,0,50,unread,,,,,,,,,,,,,,,,,
82,56,unread,,FALSE,,FALSE,,,2,,0,-1,,unread,,,,,,,,,,,,,,,,,
83,57,unread,,FALSE,,FALSE,,,1,,0,10,1,unread,5,,10,,,,,,,,,,,,,,
84,58,unread,,FALSE,,FALSE,,,1,,0,10,20,unread,,,,,,,,,,,,,,,,,
85,59,unread,,FALSE,,FALSE,,,1,,0,10,10,unread,,,,,,,,,,,,,,,,,
86,60,unread,T_29_W_WSS_FAMAS_D,TRUE,,FALSE,2,Icon_WSS_FAMAS_Asianoni,1,,0,-1,1,unread,,,,,,,,,,,,,,,,,
87,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,15,unread,,,,,,,,,,,,,,,,,
88,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,18,unread,,,,,,,,,,,,,,,,,
89,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,21,unread,,,,,,,,,,,,,,,,,
90,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,24,unread,,,,,,,,,,,,,,,,,
91,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,0,unread,,,,,,,,,,,,,,,,,
92,61,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,70,unread,,,,,,,,,,,,,,,,,
93,62,unread,,FALSE,,FALSE,,,1,,0,5,25,unread,,,,,,,,,,,,,,,,,
94,62,unread,,FALSE,,FALSE,,,1,,0,5,30,unread,,,,,,,,,,,,,,,,,
95,62,unread,,FALSE,,FALSE,,,1,,0,5,35,unread,,,,,,,,,,,,,,,,,
96,62,unread,,FALSE,,FALSE,,,1,,0,5,40,unread,,,,,,,,,,,,,,,,,
97,62,unread,,FALSE,,FALSE,,,1,,0,5,45,unread,,,,,,,,,,,,,,,,,
98,62,unread,,FALSE,,FALSE,,,1,,0,3,60,unread,,,,,,,,,,,,,,,,,
99,63,unread,,FALSE,,FALSE,,,2,,0,-1,50,unread,,,,,,,,,,,,,,,,,
100,64,unread,,FALSE,,FALSE,,,2,,0,-1,20,unread,10,unread,,,,,,,,,,,,,,,
101,65,unread,,FALSE,,FALSE,,,1,,0,-1,1201,unread,,,,,,,,,,,,,,,,,
102,67,unread,,FALSE,,FALSE,,,1,,0,4,4,unread,,,,,,,,,,,,,,,,,
103,67,unread,,FALSE,,FALSE,,,1,,0,6,4,unread,,,,,,,,,,,,,,,,,
104,67,unread,,FALSE,,FALSE,,,1,,0,8,4,unread,,,,,,,,,,,,,,,,,
105,67,unread,,FALSE,,FALSE,,,1,,0,10,4,unread,,,,,,,,,,,,,,,,,
106,67,unread,,FALSE,,FALSE,,,1,,0,12,4,unread,,,,,,,,,,,,,,,,,
107,67,unread,,FALSE,,FALSE,,,1,,0,15,4,unread,,,,,,,,,,,,,,,,,
108,66,unread,,FALSE,,FALSE,,,1,,0,-1,,,,,,,,,,,,,,,,,,,
109,68,unread,T_30_W_WSS_UMP_D,TRUE,,FALSE,2,Icon_WSS_UMP_Booyahday21,1,,100,-1,1,unread,,,,,,,,,,,,,,,,,
110,70,unread,,FALSE,,FALSE,,,1,,0,0,35,unread,,,200,unread,,,,,,,,,,,,,
111,70,unread,,FALSE,,FALSE,,,1,,0,0,0,0,-1,unread,,,,,,,,,,,,,,,
112,69,unread,,FALSE,,FALSE,,,0,,0,2,,,,,,,,,,,,,,,,,,,
113,21,unread,,TRUE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,-1,5,unread,,,,,VFX_INGAME_WSS_AK47DRAGON,,,,,,,,,,,,
114,72,unread,,FALSE,,FALSE,,,1,,0,-1,,unread,,,,,,,,,,,,,,,,,
115,71,unread,,FALSE,,FALSE,,,2,TRUE,0,0,,,unread,,,,,,,,,,,,,,,,
116,73,unread,T_33_PD_WSS_MP5_D,TRUE,,FALSE,2,Icon_WSS_MP5_Goldenwings,0,,0,-1,0.1,unread,,,,,,,,,,,,,,,,,
117,74,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,35,unread,,,,,,,,,,,,,,,,,
118,75,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,10,unread,,,,,,,,,,,,,,,,,
119,76,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,10,unread,,,,,,,,,,,,,,,,,
120,77,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,250,unread,,,,,,,,,,,,,,,,,
121,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,20,,unread,,,,,,,,,,,,,,,,,
122,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,22,,unread,,,,,,,,,,,,,,,,,
123,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,24,,unread,,,,,,,,,,,,,,,,,
124,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,26,,unread,,,,,,,,,,,,,,,,,
125,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,28,,unread,,,,,,,,,,,,,,,,,
126,78,unread,,FALSE,,FALSE,,Icon_WSS_Ak47_Dragon,0,,0,30,,unread,,,,,,,,,,,,,,,,,
127,35,unread,T_33_YS_CS_SHOP_BUFF_1_DESC,FALSE,VFX_Ingame_LwBuffAttack,TRUE,,FF_UI_shopbuff_attack01_white,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
128,36,unread,T_33_YS_CS_SHOP_BUFF_2_DESC,FALSE,VFX_Ingame_LwBuffAttack,TRUE,,FF_UI_shopbuff_attack04_white,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
129,37,unread,T_33_YS_CS_SHOP_BUFF_3_DESC,FALSE,VFX_Ingame_LwBuffAttack,TRUE,,FF_UI_shopbuff_attack03_white,2,TRUE,0,-1,30,unread,,,,,,,,,,,,,,,,,
130,64,unread,T_33_YS_CS_SHOP_BUFF_4_DESC,FALSE,VFX_Ingame_LwBuffAttack,TRUE,,FF_UI_shopbuff_attack02_white,2,TRUE,0,-1,10,unread,10,unread,,,,,,,,,,,,,,,
131,48,unread,T_33_YS_CS_SHOP_BUFF_5_DESC,FALSE,VFX_Ingame_LwBuffDefense,TRUE,,FF_UI_shopbuff_defense03_white,2,TRUE,0,-1,,unread,,,,,,,,,,,,,,,,,
132,49,unread,T_33_YS_CS_SHOP_BUFF_6_DESC,FALSE,VFX_Ingame_LwBuffDefense,TRUE,,FF_UI_shopbuff_defense04_white,2,TRUE,0,-1,1000,unread,,,,,,,,,,,,,,,,,
133,50,unread,T_33_YS_CS_SHOP_BUFF_7_DESC,FALSE,VFX_Ingame_LwBuffDefense,TRUE,,FF_UI_shopbuff_defense02_white,2,TRUE,0,-1,,unread,,,,,,,,,,,,,,,,,
134,53,unread,T_33_YS_CS_SHOP_BUFF_8_DESC,FALSE,VFX_Ingame_LwBuffDefense,TRUE,,FF_UI_shopbuff_defense01_white,2,TRUE,0,-1,25,unread,,,,,,,,,,,,,,,,,
135,39,unread,T_33_YS_CS_SHOP_BUFF_9_DESC,FALSE,VFX_Ingame_LwBuffOther,TRUE,,FF_UI_shopbuff_other01_white,2,TRUE,0,-1,1,unread,,,,,,,,,,,,,,,,,
136,79,unread,T_33_YS_CS_SHOP_BUFF_10_DESC,FALSE,VFX_Ingame_LwBuffOther,TRUE,,FF_UI_shopbuff_other03_white,2,TRUE,0,0,150,unread,,,,,,,,,,,,,,,,,
137,52,unread,T_33_YS_CS_SHOP_BUFF_11_DESC,FALSE,VFX_Ingame_LwBuffOther,TRUE,,FF_UI_shopbuff_other02_white,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
138,63,unread,T_33_YS_CS_SHOP_BUFF_12_DESC,FALSE,VFX_Ingame_LwBuffOther,TRUE,,FF_UI_shopbuff_other04_white,2,TRUE,0,-1,30,unread,,,,,,,,,,,,,,,,,
139,80,unread,,FALSE,,FALSE,,,0,TRUE,0,60,30,unread,,,,,,,,,,,,,,,,,
140,35,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,30,unread,,,,,,,,,,,,,,,,,
141,36,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,80,unread,,,,,,,,,,,,,,,,,
142,37,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
143,38,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
144,39,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,100,unread,,,,,,,,,,,,,,,,,
145,41,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,40,unread,,,,,,,,,,,,,,,,,
146,42,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,40,unread,,,,,,,,,,,,,,,,,
147,43,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,40,unread,,,,,,,,,,,,,,,,,
148,44,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
149,45,unread,,FALSE,,FALSE,,,2,TRUE,0,0,200,unread,,,,,,,,,,,,,,,,,
150,46,unread,,FALSE,,FALSE,,,2,TRUE,0,0,40,unread,,,,,,,,,,,,,,,,,
151,47,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,500,unread,,,,,,,,,,,,,,,,,
152,49,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,600,unread,,,,,,,,,,,,,,,,,
153,50,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,,unread,,,,,,,,,,,,,,,,,
154,52,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,80,unread,,,,,,,,,,,,,,,,,
155,53,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,80,unread,,,,,,,,,,,,,,,,,
156,54,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,50,unread,,,,,,,,,,,,,,,,,
157,55,unread,,FALSE,,FALSE,,,2,TRUE,0,0,50,unread,,,,,,,,,,,,,,,,,
158,55,unread,,FALSE,,FALSE,,,2,TRUE,0,0,100,unread,,,,,,,,,,,,,,,,,
159,56,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,,unread,,,,,,,,,,,,,,,,,
160,81,unread,T_34_PD_FlagBattle_TIPS_11,FALSE,,TRUE,,ui_icon_skill_Flagbattle,1,,0,30,20,unread,20,unread,5,unread,161,,,,,,,,,,,,
161,82,unread,T_34_PD_FlagBattle_TIPS_11,FALSE,,TRUE,,ui_icon_skill_Flagbattle,1,,0,30,30,unread,40,unread,10,unread,162,,,,,,,,,,,,
162,83,unread,T_34_PD_FlagBattle_TIPS_11,FALSE,,TRUE,,ui_icon_skill_Flagbattle,1,,0,-1,50,unread,60,unread,50,unread,,,,,,,,,,,,,
163,84,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,3000,unread,,,,,,,,,,,,,,,,,
164,85,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,10000,unread,5,unread,,,,,,,,,,,,,,,
165,86,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,25,unread,0.3,unread,0.25,unread,0.6,,,,,,,,,,,,
166,67,unread,,FALSE,,FALSE,,,1,,0,-1,20,unread,,,,,,,,,,,,,,,,,
167,63,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,100,unread,,,,,,,,,,,,,,,,,
168,89,unread,T_35_JL_1887_SKILL_D,TRUE,,FALSE,1,Icon_WSS_M1887_Digitaluniverse,4,,0,1.5,-0.025,unread,,,,,,,,,,,,,,,,,
169,88,unread,,FALSE,,FALSE,,,0,,0,-1,0.3,unread,,,,,,,,,,,,,,,,,
170,87,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,10,unread,1,unread,1,unread,,,,,,,,,,,,,
171,87,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,10,unread,1.5,unread,1,unread,,,,,,,,,,,,,
172,87,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,10,unread,2.5,unread,1,unread,,,,,,,,,,,,,
173,91,unread,,FALES,,FALSE,,,1,,0,5,3,unread,0.06,unread,-0.09,unread,,,,,,,,,,,,,
174,92,unread,,FALES,,FALSE,,,1,,0,-1,1,unread,200,unread,6,unread,,,,,,,,,,,,,
777,3,unread,T_35_QL_CSBUFF_ICONDEC_2,TRUE,EFFECT_CSBUFF_HPRECOVER,TRUE,,FF_UI_CS_Buff_Recovery,1,,0,2000,20000,unread,,,,,,,,,,,,,,,,,
175,3,unread,,FALSE,,FALSE,,,1,,0,4,200,unread,,,,,,,,,,,,,,,,,
176,3,unread,T_35_QL_CSBUFF_ICONDEC_1,TRUE,EFFECT_CSBUFF_HPDECREASE,TRUE,,FF_UI_CS_Buff_BloodLoss,1,,0,2000,-4000,unread,,,,,,,,,,,,,,,,,
177,3,unread,T_35_QL_CSBUFF_ICONDEC_2,TRUE,EFFECT_CSBUFF_HPRECOVER,TRUE,,FF_UI_CS_Buff_Recovery,1,,0,2000,30000,unread,,,,,,,,,,,,,,,,,
178,90,unread,T_35_QL_CSBUFF_ICONDEC_3,TRUE,EFFECT_CSBUFF_HPRECOVER,TRUE,,FF_UI_CS_Buff_BloodImprove,1,,0,0,125,unread,,,,,,,,,,,,,,,,,
179,86,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,35,unread,0.5,unread,0.3,unread,0.6,,,,,,,,,,,,
180,92,unread,,FALES,,FALSE,,,1,,0,-1,1,unread,200,unread,6,unread,,,,,,,,,,,,,
181,38,unread,,FALSE,,FALSE,,,2,TRUE,0,-1,10,unread,,,,,,,,,,,,,,,,,
182,21,unread,T_24_O_WSS_AK47_D,TRUE,,FALSE,2,Icon_WSS_Ak47_Dragon,0,,0,-1,5,unread,,,,,VFX_INGAME_WSS_AK47DRAGON,,,,,,,,,,,,
183,60,unread,T_29_W_WSS_FAMAS_D,TRUE,,FALSE,2,Icon_WSS_FAMAS_Asianoni,1,,0,-1,1,unread,,,,,,,,,,,,,,,,,
184,93,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,10,unread,-1,unread,,,,,,,,,,,,,,,
185,94,unread,,FALSE,,FALSE,,,1,TRUE,0,-1,10,unread,-1,unread,,,,,,,,,,,,,,,
186,97,unread,,FALSE,,FALSE,,,0,TRUE,0,-1,,,,,,,,,,,,,,,,,,,
187,98,unread,,FALSE
