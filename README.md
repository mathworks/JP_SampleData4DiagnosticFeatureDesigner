# DEMO: アプリを使用した特徴量調査

このプログラムでは、デモに使用するデータファイル sampleData.mat を作成します。もともとの1 日当たり 6 秒間の振動信号を連続 50 日間取得したものですが、ここでは計算量軽減のため、1 日当たり 1 秒間、1 日おき 25 日分のデータに削減します。

# Dataset

20 歯のピニオンギヤで駆動する 2 MW 風力タービン高速シャフトから収集されたデータを使用します[1]。まず、[https://github.com/mathworks/WindTurbineHighSpeedBearingPrognosis-Data](https://github.com/mathworks/WindTurbineHighSpeedBearingPrognosis-Data) からリポジトリ全体を zip ファイルとしてダウンロードして、本スクリプトと同じディレクトリに保存してください。以下のコマンドを使用してファイルを解凍します。こちらのデータでは計測間隔は 1 日間隔です。

```matlab:Code
if exist('WindTurbineHighSpeedBearingPrognosis-Data-main.zip', 'file')
    unzip('WindTurbineHighSpeedBearingPrognosis-Data-main.zip')
end
```

# Data Import

まず風力タービンデータに対して fileDatastore を作成します。使用するデータには振動とタコメータの信号が含まれています。各 mat ファイルからcutomreader 関数で読み取りますが、1 秒分の振動データに加えてファイル名から日付情報も合わせて取得します。

```matlab:Code
dir = 'WindTurbineHighSpeedBearingPrognosis-Data-main';
ds = fileDatastore(fullfile('.', dir,'*.mat'),'ReadFcn',@customreader,'UniformRead',true);
data = readall(ds)
```

| |Date|vibration|
|:--:|:--:|:--:|
|1|2013/03/07|97656x1 timetable|
|2|2013/03/08|97656x1 timetable|
|3|2013/03/09|97656x1 timetable|
|4|2013/03/10|97656x1 timetable|
|5|2013/03/11|97656x1 timetable|
|6|2013/03/12|97656x1 timetable|
|7|2013/03/13|97656x1 timetable|
|8|2013/03/14|97656x1 timetable|
|9|2013/03/15|97656x1 timetable|
|10|2013/03/16|97656x1 timetable|
|11|2013/03/17|97656x1 timetable|
|12|2013/03/17|97656x1 timetable|
|13|2013/03/18|97656x1 timetable|
|14|2013/03/20|97656x1 timetable|

# Data Reduction

ここでは計算量軽減のため、さらに1 日おき 25 日分のデータに削減します。

```matlab:Code
sampleData = data;
sampleData(1:2:end,:) = [];
```

さらに前半のデータには faultCode = 0、後半のデータについては faultcode = 1 とラベル付けします。

```matlab:Code
sampleData.faultcode = [zeros(13,1); ones(12,1)];
save('sampleData.mat','sampleData');
```

任意に変更して、他にもアプリで使用するサンプルデータを作成して試してみてください。

# References

[1] [<http://data-acoustics.com/measurements/bearing-faults/bearing-3/](http://data-acoustics.com/measurements/bearing-faults/bearing-3/)> Bechhoefer, Eric, Brandon Van Hecke, and David He. "Processing for improved spectral analysis." *Annual Conference of the Prognostics and Health Management Society, New Orleans, LA, Oct*. 2013.

# Helper Function

```matlab:Code
function data = customreader(filename)
% Read variables for the fileEnsemble
% Inputs:
% filename  - a string of the mat file name to read from.
% Output:
% data      - return a table with a single row
% Copyright 2018 The MathWorks, Inc.

data = table;
% Extract the datetime information from the file names
[~, fname] = fileparts(filename);
token = regexp(fname, 'data-(\d+)', 'tokens');
data.Date = datetime(token{1}{1}, 'InputFormat', 'yyyyMMdd');

% Load the vibration signal from the mat file
mfile = matfile(filename); % Allows partial loading
vib = mfile.vibration;

% Extract the first 1s of signal and convert it to timetable
fs = 97656; % Sampling rate
tVibration = timetable(vib(1:fs),'SampleRate',fs);
data.vibration = {tVibration};

end

```

*Copyright 2018-2022 The MathWorks, Inc.*
