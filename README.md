# US-Vevo
%Extract data from Vevo files to Matlab
close all
clear all
clc

%///user defined\\\
ModeName = '.bmode';

files = dir('*.raw.xml');
NFiles = size(files,1);

for ifile = 1:NFiles
     % for ifile = 1:1 
        fnameBase =  files(ifile).name(1:end-4);
        fnameXml = [fnameBase '.xml'];
        param = VsiParseXml(fnameXml, '.pamode');
        %///not used with 2d\\\
%         PaScanDistance = param.PaScanDistance; %mm
%         PaStepSize = param.PaStepSize; %mm
%         NFrames = round(PaScanDistance/PaStepSize);
        NFrames = 5;
        [notused, PAWidthAxis, PADepthAxis] = VsiOpenRawPa(fnameBase, '.pamode', 1);
        PADepthAxis = PADepthAxis(1:end-10);
        PAWidthAxis = PAWidthAxis - mean(PAWidthAxis);
                   
    for iframe = 1:NFrames
        [Rawdata, WidthAxis, DepthAxis] = OpenRaw(fnameBase, ModeName, iframe);
        WidthAxis = WidthAxis - mean(WidthAxis);
        [y,left] = min(abs(WidthAxis - min(PAWidthAxis)));
        [y,right] = min(abs(WidthAxis - max(PAWidthAxis)));
        [y,top] = min(abs(DepthAxis - min(PADepthAxis)));
        top = top-1;
        [y,bottom] = min(abs(DepthAxis - max(PADepthAxis)));
        US(:,:,iframe) = Rawdata(top:bottom,left:right);
        US_small(:,:,iframe) = interp2(WidthAxis(left:right),DepthAxis(top:bottom),US(:,:,iframe),PAWidthAxis,PADepthAxis');
     
    end
    
    US_small = uint8(US_small);
    fnamesave = files(ifile).name(1:end-8);
    save([num2str(fnamesave) '_US.mat'], 'US', 'US_small', 'WidthAxis', 'DepthAxis', 'PAWidthAxis', 'PADepthAxis')
    clear('US','US_small')

end
    
