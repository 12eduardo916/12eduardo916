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
