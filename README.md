# matlab-project
classdef KangWan < matlab.apps.AppBase

    % Properties that correspond to app components
    properties (Access = public)
        UIFigure              matlab.ui.Figure
        EditField_15Label_21  matlab.ui.control.Label
        EditField_15Label_20  matlab.ui.control.Label
        EditField_15Label_19  matlab.ui.control.Label
        EditField_15Label_18  matlab.ui.control.Label
        Button_5              matlab.ui.control.StateButton
        Button_4              matlab.ui.control.StateButton
        EditField_15Label_17  matlab.ui.control.Label
        UITable               matlab.ui.control.Table
        EditField_15Label_15  matlab.ui.control.Label
        EditField_15          matlab.ui.control.EditField
        Label_9               matlab.ui.control.Label
        EditField_14          matlab.ui.control.EditField
        EditField_13          matlab.ui.control.EditField
        Label_8               matlab.ui.control.Label
        Label_13              matlab.ui.control.Label
        EditField_9           matlab.ui.control.EditField
        Label_11              matlab.ui.control.Label
        EditField_8           matlab.ui.control.EditField
        Label_10              matlab.ui.control.Label
        EditField_7           matlab.ui.control.EditField
        Label_7               matlab.ui.control.Label
        EditField_3           matlab.ui.control.EditField
        Label_3               matlab.ui.control.Label
        EditField_2           matlab.ui.control.EditField
        Label_2               matlab.ui.control.Label
        EditField             matlab.ui.control.EditField
        Label                 matlab.ui.control.Label
        EditField_15Label_5   matlab.ui.control.Label
        EditField_15Label_4   matlab.ui.control.Label
        EditField_15Label_3   matlab.ui.control.Label
        TextArea              matlab.ui.control.TextArea
        Image                 matlab.ui.control.Image
        EditField_15Label_2   matlab.ui.control.Label
        Button_3              matlab.ui.control.StateButton
        Button_2              matlab.ui.control.StateButton
        Button                matlab.ui.control.StateButton
        UIAxes                matlab.ui.control.UIAxes
    end

    
    properties (Access = private)
        filename %文件名
        string_array %有效文件sheet名
        values_last_zhuan_mean %上升段位移值
        output_excel_file %新Excel表
        up_load %上升段荷载值
    end
    

    % Callbacks that handle component events
    methods (Access = private)

        % Value changed function: Button
        function ButtonValueChanged(app, event)
            value = app.Button.Value;
          % 提示用户选择要打开的文件
[file, path] = uigetfile('*.xlsx', '选择要打开的Excel文件');

% 检查用户是否取消了选择
if isequal(file, 0)
    errordlg('您取消了选择文件。','错误');
    return
else
    % 创建文件路径
    app.filename = fullfile(path, file);
    errordlg('成功读取Excel文件的数据。','成功');
end

Table_sheet_group_name = {'1', '2', '3'};
Unit_group_number = length(Table_sheet_group_name); 
Maximum = zeros(1, Unit_group_number);
Corresponding_values = zeros(1, Unit_group_number);

Storage_of_maximum_value_in_the_row=zeros(1,3); 
Storage_of_number_rows=zeros(1,3);
Storage_of_first_column=cell(1, 3);
Storage_of_second_column=cell(1,3);
for i = 1:Unit_group_number
    % 读取工作excel工作表数据
    worksheet_data = xlsread(app.filename, Table_sheet_group_name{i});
    
    % 获取第一列和第二列数据
    First_column = worksheet_data(:, 1);
    Second_column = worksheet_data(:, 2);
    num_rows(i) = length(worksheet_data);
    
    % 找到第一列数据的最大值及其索引
    [max_value, max_index(i)] = max(First_column);
       
    % 找到第二列数据的最大值
    [Maximum_value_of_the_second_column_of_data, max_index2(i)] = max(Second_column);

    % 获取对应最大值的第二列数据
    corresponding_value = Second_column(max_index(i));
    
    % 存储结果
    Storage_of_maximum_value_in_the_row(1,i)=max_index(i);
    max_index_cunchu2(1,i)=max_index2(i);
    Maximum(i) = max_value;
    max_values2(i) = Maximum_value_of_the_second_column_of_data;
    Corresponding_values(i) = corresponding_value;
    Storage_of_number_rows(1,i)=num_rows(i);
    Storage_of_first_column{i}=First_column;
    Storage_of_second_column{i}=Second_column;
end

% 计算最大值和对应值的平均值
Maximum_average = mean(Maximum);
average_corresponding_value = mean(Corresponding_values);

% 循环遍历数据并分配给编辑字段
for i = 1:Unit_group_number
       
    % 将数据显示在编辑字段中
    app.EditField.Value = num2str(Maximum(1));
     app.EditField_2.Value = num2str(Maximum(2));
      app.EditField_3.Value = num2str(Maximum(3));
             app.EditField_7.Value = num2str(Corresponding_values(1));
     app.EditField_8.Value = num2str(Corresponding_values(2));
      app.EditField_9.Value = num2str(Corresponding_values(3));
                     app.EditField_13.Value = num2str(Maximum_average);
         app.EditField_14.Value = num2str(average_corresponding_value);
end
%step2 试验数据有效性判断
desirability_1=zeros(1,3);
s=0;
% 创建一个新的Excel文件
app.output_excel_file = [app.filename, '_有效.xlsx'];
for i = 1:Unit_group_number

x = Maximum(i);                %最大应力
y = Maximum_average;            %最大应力均值
z = average_corresponding_value;  %最大应力对应应变均值

% 定义有效范围
y_lower_limit = 0.85 * x;
y_upper_limit = 1.15 * x;

% 判断有效性

valid = (y_lower_limit <= y && y <= y_upper_limit);

% 根据判断结果输出
if valid
    s=s+1;
New_workgroup_name = ['', Table_sheet_group_name{i}];
% 读取当前工作表的数据
data = xlsread(app.filename, Table_sheet_group_name{i});
% 写入新Excel文件的新工作表
xlswrite(app.output_excel_file, data, New_workgroup_name);

% 指定要创建的元胞数组的大小
info_cell_array = cell(s, 1);

for q = 1:s
    info = sprintf('%d', q);
    info_cell_array{q} = info;
end
j=0;
else
    fprintf('第%s组数据无效\n',Table_sheet_group_name{i});
    j=j+1;
end
desirability_1(1,i)=Maximum(i);
end
if s >= 2  
    %fprintf('试验结果有效');
    msgbox("试验结果有效,请稍后...",'提示','help');
else
    %fprintf('试验结果无效');
    msgbox("试验结果无效，请核实后重试",'提示','help');
    return
end
%step3 有效荷载-位移曲线上升段荷载归一处理，下降段位移归一处理

% 获取Excel文件中的所有工作表名称

num_sheets_new = length(info_cell_array);
[status,sheets] = xlsfinfo(app.output_excel_file);

% 初始化一个新的单元数组，根据 s 的值确定大小
A_new_unit_array = sheets(1, s);

% 读取第2到第s个值并存储到新数组中
for i = 2:s+1
    A_new_unit_array{i-1} = sheets{i};
end

% 显示新的单元数组
disp(A_new_unit_array);
% 将单元数组转换为字符串数组
app.string_array = cellstr(A_new_unit_array);
desirability_new=zeros(1,s);
New_maximum_index_storage=zeros(1,s);  
number_rowsnew_storage=zeros(1,s);
for i = 1:num_sheets_new
    % 读取工作表数据
    Read_new_worksheet_data = xlsread(app.output_excel_file, app.string_array{i});  
    
    % 获取第一列和第二列数据
    First_column_new = Read_new_worksheet_data(:, 1);
    Second_column_new = Read_new_worksheet_data(:, 2);
    num_rows_new(i) = length(Read_new_worksheet_data);
    
    % 找到第一列数据的最大值及其索引
    [max_value_new, max_index_new(i)] = max(First_column_new);
       
    % 找到第二列数据的最大值
    [max_value2_new, max_index2_new(i)] = max(Second_column_new);

    % 获取对应最大值的第二列数据
    corresponding_value_new = Second_column_new(max_index_new(i));
    
    % 存储结果
    New_maximum_index_storage(1,i)=max_index_new(i);
    max_index_new_cunchu2(1,i)=max_index2_new(i);
    max_values_new(i) = max_value_new;
    max_values2_new(i) = max_value2_new;
    corresponding_values_new(i) = corresponding_value_new;
    number_rows_new_cunchu(1,i)=num_rows_new(i);

    desirability_new(1,i)=max_values_new(i);
end

% 初始化一个空单元数组来存储数据
An_empty_array_of_cells = cell(1, s); 
An_empty_array_of_cells_1 = cell(1, s);
An_empty_array_of_cells_2 = cell(1, s);
for i = 1:num_sheets_new
% 读取Excel文件第二列数据 

data2 = xlsread(app.output_excel_file,app.string_array{i}, 'B:B');   %读取sheet中第二列的值

the_first_column_corresponds_max_to_last = data2(max_index_new(1,i):num_rows_new(1,i));       %sheet中第一列最大值对应第二列的值到第二列最后的值

the_first_column_corresponds__1 = data2(1:max_index_new(1,i));        %sheet中第二列第一个值到峰值对于的值

An_empty_array_of_cells{i} =the_first_column_corresponds_max_to_last;
% 归一化处理
Normalization_processing_2 = An_empty_array_of_cells{i} / corresponding_values_new(i);  
An_empty_array_of_cells_1{i} =Normalization_processing_2;
% 下降段位移归一处理
Normalization_of_displacement_in_the_descending_section = [];  
for j = 1:length(Normalization_processing_2)
    if Normalization_processing_2(j) >= 1 && Normalization_processing_2(j) < 1.2
        rounded_value_up = round(Normalization_processing_2(j), 2);
    elseif Normalization_processing_2(j) >= 1.2
        rounded_value_up = round(Normalization_processing_2(j), 1);
    else
        %fprintf('数 %.2f 超出范围，将被忽略。\n', input_data(j));
        continue;
    end
    Normalization_of_displacement_in_the_descending_section = [Normalization_of_displacement_in_the_descending_section,rounded_value_up];
    Output_Result_Transfer_to_Line = Normalization_of_displacement_in_the_descending_section';  
    An_empty_array_of_cells_2{i} =Output_Result_Transfer_to_Line;
end

% 初始化变量来存储最多列数据的列和列数据数量
Max_column_date = [];
max_element_count = 0;

% 遍历单元数组中的每列数据
for i = 1:numel(An_empty_array_of_cells_2)
    % 获取当前列的元素数量
    the_number_current_element_count = numel(An_empty_array_of_cells_2{i});
    
    % 检查是否当前列的元素数量更多
    if the_number_current_element_count > max_element_count
        max_element_count = the_number_current_element_count;
        Max_column_date = An_empty_array_of_cells_2{i};
    end
end

%下降段
% 提取第一列和第二列数据
col3 = Max_column_date;

% 找到第一列中的唯一值
Unique_values_in_the_first_column = unique(col3); 

end
 
To_the_last_value=zeros(s,128);  
To_the_last_value_up=zeros(s,length(Unique_values_in_the_first_column));

Empty_cell_array_3 = cell(1, s);  

for i = 1:num_sheets_new
% 读取Excel文件第一列数据 
data3 = xlsread(app.output_excel_file, app.string_array{i}, 'A:A');  %读取sheet中第一列的值

values_to_max = data3(1:max_index_new(1,i));           %sheet中第一列第一个值到峰值

values_to_max_up = data3(max_index_new(1,i):num_rows_new(1,i));   %sheet中第一列峰值到最后的值
Empty_cell_array_3{i} = values_to_max_up;

% 初始化变量来存储最多列数据的列和列数据数量
max_column_1 = [];
max_element_count_1 = 0;
% 遍历单元数组中的每列数据
for l = 1:numel(Empty_cell_array_3)
    % 获取当前列的元素数量
    current_element_count_1 = numel(Empty_cell_array_3{l});
    
    % 检查是否当前列的元素数量更多
    if current_element_count_1 > max_element_count_1
        max_element_count_1 = current_element_count_1;
        max_column_1 = Empty_cell_array_3{l};
    end
end

end

for i = 1:num_sheets_new
% 读取Excel文件第一列数据 
data3 = xlsread(app.output_excel_file, app.string_array{i}, 'A:A');  %读取sheet中第一列的值

values_to_max = data3(1:max_index_new(1,i));           %sheet中第一列第一个值到峰值

values_to_max_up = data3(max_index_new(1,i):num_rows_new(1,i));   %sheet中第一列峰值到最后的值

data4 = xlsread(app.output_excel_file, app.string_array{i}, 'B:B');   %读取sheet中第二列的值

the_first_column_corresponds_max_to_last = data4(max_index_new(1,i):num_rows_new(1,i));       %sheet中第一列峰值对应第二列的位移值到第二列最后的值

the_first_column_corresponds__1 = data4(1:max_index_new(1,i));        %sheet中第二列第一个值到荷载峰值对于的位移值
% 归一化处理
Normalization_processing_data3 = values_to_max / desirability_new(1,i);
Normalization_processing_data4 = the_first_column_corresponds_max_to_last / corresponding_values_new(i);
% 遍历输入的数
% 上升段荷载归一处理   
Normalization_of_ascending_section_load = [];
for j = 1:length(Normalization_processing_data3)
    if Normalization_processing_data3(j) >= 0 && Normalization_processing_data3(j) < 0.97
        rounded_value = round(Normalization_processing_data3(j), 2);
    elseif Normalization_processing_data3(j) >= 0.97 && Normalization_processing_data3(j) <= 1
        rounded_value = round(Normalization_processing_data3(j), 3);
    else
       % fprintf('数 %.2f 超出范围，将被忽略。\n', input_data(j));
        continue;
    end
    Normalization_of_ascending_section_load = [Normalization_of_ascending_section_load,rounded_value];
    Normalization_of_displacement_descending_zhuan = Normalization_of_ascending_section_load';
end

% 下降段位移归一处理 
Normalization_of_displacement_descending = [];
for j = 1:length(Normalization_processing_data4)
    if Normalization_processing_data4(j) >= 1 && Normalization_processing_data4(j) < 1.2
        rounded_value_up_1 = round(Normalization_processing_data4(j), 2);
    elseif Normalization_processing_data4(j) >= 1.2
        rounded_value_up_1 = round(Normalization_processing_data4(j), 1);
    else
       % fprintf('数 %.2f 超出范围，将被忽略。\n', input_data(j));
        continue;
    end
    Normalization_of_displacement_descending = [Normalization_of_displacement_descending,rounded_value_up_1];
    output_data_zhuan_up_1 = Normalization_of_displacement_descending';
end

%step4缩减并对有效荷载-位移曲线上升段和下降段归一曲线数据求平均

%上升段

% 提取第一列和第二列数据
col1 = Normalization_of_displacement_descending_zhuan;
col2 = the_first_column_corresponds__1;

% 找到第一列中的唯一值
unique_values = unique(col1);

% 初始化存储平均值的数组
average_values = [];

% 计算每个相同值对应第二列值的平均值
for k = 1:length(unique_values)
    indices = find(col1 == unique_values(k));
    avg_value = mean(col2(indices));
    average_values = [average_values, avg_value];
end
average_values_zhuan=average_values';

To_the_last_value(i,1:length(average_values))=average_values;
values_last_zhuan=To_the_last_value';
app.values_last_zhuan_mean=mean(values_last_zhuan,2);
%下降段

% 遍历单元数组中的每列数据
for l = 1:numel(Empty_cell_array_3)
    % 获取当前列的元素数量
    current_element_count_1 = numel(Empty_cell_array_3{l});
    
    % 检查是否当前列的元素数量更多
    if current_element_count_1 > max_element_count_1
        max_element_count_1 = current_element_count_1;
        max_column_1 = Empty_cell_array_3{l};
    end
end

% 提取第一列和第二列数据
col3 = Max_column_date;
col4 = max_column_1;

% 找到第一列中的唯一值
Unique_values_in_the_first_column = unique(col3);

% 初始化存储平均值的数组
average_values_up = [];

% 计算每个相同值对应第二列值的平均值
for k = 1:length(Unique_values_in_the_first_column)
    indices_up = find(col3 == Unique_values_in_the_first_column(k));
    avg_value_up = mean(col4(indices_up));
    average_values_up = [average_values_up, avg_value_up];
end
average_values_zhuan_up=average_values_up';

To_the_last_value_up(i,1:length(average_values_up))=average_values_up;
values_last_zhuan_up=To_the_last_value_up';

values_last_zhuan_mean_up=zeros(size(values_last_zhuan_up, 1), 1);  % 初始化存储平均值的数组
for i = 1:size(values_last_zhuan_up, 1)
    row = values_last_zhuan_up(i, :);  % 获取当前行的数据
    nonzero_elements = row(row ~= 0);  % 获取非零元素
    if isempty(nonzero_elements)
        values_last_zhuan_mean_up(i) = 0;  % 如果当前行都是0，则平均值为0
    else
        values_last_zhuan_mean_up(i) = mean(nonzero_elements);  % 计算非零元素的平均值
    end
end

end
app.up_load=unique_values*y;
down_weiyi=Unique_values_in_the_first_column*z;
if length(app.values_last_zhuan_mean) ==length(app.up_load)

plot(app.UIAxes,app.values_last_zhuan_mean, app.up_load,'r-','LineWidth',2);
hold(app.UIAxes,"on");
plot(app.UIAxes,down_weiyi, values_last_zhuan_mean_up,'g-','LineWidth',2);
% 写入新Excel文件的Sheet1工作表
xlswrite(app.output_excel_file, app.values_last_zhuan_mean, 1,'A1');
xlswrite(app.output_excel_file, app.up_load, 1,'B1');
xlswrite(app.output_excel_file, values_last_zhuan_mean_up, 1,'B129');
xlswrite(app.output_excel_file, down_weiyi, 1,'A129');

for i=1:Unit_group_number
    hold(app.UIAxes,"on");
    plot(app.UIAxes,Storage_of_second_column{i},Storage_of_first_column{i},'LineWidth',1);
end
% 添加图例
legend(app.UIAxes,'上升段', '下降段', '数据 1', '数据 2', '数据 3');
 xlabel('应变');
 ylabel('应力(Mpa)');
 title('混凝土轴心抗拉应力-应变全曲线');
app.EditField_15.Value = cell2mat(app.string_array);
     else
        msgbox("实验数据过少不满足归一化要求，请核实",'提示','help');
    return
end
        end

        % Value changed function: EditField
        function EditFieldValueChanged(app, event)

        end

        % Value changed function: Button_2
        function Button_2ValueChanged(app, event)
            value = app.Button_2.Value;
msgbox("1.请确保读取的Excel表格中有4组数据，并且将每组数据分别存放在4个sheet中，并以阿拉伯数字1234分别命名，否则将读取出错。" + ...
    "2.每个sheet中第一列为应力值，第二列为应变值 ",'使用须知','help');
        end

        % Value changed function: Button_3
        function Button_3ValueChanged(app, event)
            value = app.Button_3.Value;
            choice=questdlg('您要关闭吗？','关闭','Yes','No','No');
            switch choice
                case 'Yes'
                    delete(app.UIFigure);
                    return;
                case 'No'
                    return;
            end
        end

        % Value changed function: EditField_15
        function EditField_15ValueChanged(app, event)
            value = app.EditField_15.Value;             
        end

        % Display data changed function: UITable
        function UITableDisplayDataChanged(app, event)
    
        end

        % Cell edit callback: UITable
        function UITableCellEdit(app, event)
            indices = event.Indices;
            newData = event.NewData;
            
        end

        % Value changed function: Button_4
        function Button_4ValueChanged(app, event)
            value = app.Button_4.Value;
            choice=questdlg('读取数据成功后点击（生成），未读取成功点击（返回）','提示','生成','返回','返回');
            switch choice
                case '生成'
             %读取表格
             % 使用exist函数检查文件是否存在
file_exists = exist(app.output_excel_file, 'file');

if file_exists == 2
    t=readtable(app.output_excel_file,"Sheet",1);
            app.UITable.Data=t;
                    return;
else
    msgbox("未读取数据，请先读取数据",'提示','help');
end
            
                case '返回'
                    return;
            end
           
        end

        % Callback function
        function TextArea_2ValueChanged(app, event)
       
        end

        % Callback function
        function Button_5ValueChanged(app, event)

        end

        % Value changed function: Button_5
        function Button_5ValueChanged2(app, event)
            value = app.Button_5.Value;
            choice=questdlg('读取数据成功后点击（导出），未读取成功点击（返回）','提示','导出','返回','返回');
            switch choice
                case '导出'
                    %读取表格
             % 使用exist函数检查文件是否存在
file_exists = exist(app.output_excel_file, 'file');

if file_exists == 2
            x=xlsread(app.output_excel_file, 1,'A:A');
            y=xlsread(app.output_excel_file, 1,'B:B');
            % 提示用户选择保存路径
[file, path] = uiputfile('*.xlsx', '选择保存路径和文件名');

if file ~= 0
    % 构建完整的文件路径
    fullFilePath = fullfile(path, file);

    % 将X和Y数据保存到Excel文件
    data = [x, y]; % 将X和Y数据组合成一个矩阵
    writematrix(data, fullFilePath);
    
    fprintf('数据已成功保存到 %s\n', fullFilePath);
else
    fprintf('未选择保存路径，数据未保存。\n');
end
return;
else
    msgbox("未读取数据，请先读取数据",'提示','help');
end
                case'返回'
                    return;
            end
        end
    end

    % Component initialization
    methods (Access = private)

        % Create UIFigure and components
        function createComponents(app)

            % Get the file path for locating images
            pathToMLAPP = fileparts(mfilename('fullpath'));

            % Create UIFigure and hide until all components are created
            app.UIFigure = uifigure('Visible', 'off');
            app.UIFigure.Position = [100 100 951 760];
            app.UIFigure.Name = 'MATLAB App';

            % Create UIAxes
            app.UIAxes = uiaxes(app.UIFigure);
            title(app.UIAxes, '纤维混凝土抗弯荷载-挠度全曲线')
            xlabel(app.UIAxes, '挠度（mm）')
            ylabel(app.UIAxes, '荷载(Mpa)')
            zlabel(app.UIAxes, 'Z')
            app.UIAxes.Position = [17 26 627 474];

            % Create Button
            app.Button = uibutton(app.UIFigure, 'state');
            app.Button.ValueChangedFcn = createCallbackFcn(app, @ButtonValueChanged, true);
            app.Button.Text = '读取表格';
            app.Button.Position = [9 542 121 22];

            % Create Button_2
            app.Button_2 = uibutton(app.UIFigure, 'state');
            app.Button_2.ValueChangedFcn = createCallbackFcn(app, @Button_2ValueChanged, true);
            app.Button_2.Text = '使用须知';
            app.Button_2.BackgroundColor = [1 0 0];
            app.Button_2.FontWeight = 'bold';
            app.Button_2.Position = [9 585 121 22];

            % Create Button_3
            app.Button_3 = uibutton(app.UIFigure, 'state');
            app.Button_3.ValueChangedFcn = createCallbackFcn(app, @Button_3ValueChanged, true);
            app.Button_3.Text = '退出程序';
            app.Button_3.Position = [9 499 121 22];

            % Create EditField_15Label_2
            app.EditField_15Label_2 = uilabel(app.UIFigure);
            app.EditField_15Label_2.HorizontalAlignment = 'right';
            app.EditField_15Label_2.Position = [325 544 70 18];
            app.EditField_15Label_2.Text = '组数据有效';

            % Create Image
            app.Image = uiimage(app.UIFigure);
            app.Image.Position = [9 617 121 134];
            app.Image.ImageSource = fullfile(pathToMLAPP, '校徽.jpeg');

            % Create TextArea
            app.TextArea = uitextarea(app.UIFigure);
            app.TextArea.HorizontalAlignment = 'center';
            app.TextArea.FontSize = 20;
            app.TextArea.FontColor = [1 1 1];
            app.TextArea.BackgroundColor = [0 0.4471 0.7412];
            app.TextArea.Position = [138 713 793 38];
            app.TextArea.Value = {'一种适用于纤维混凝土抗弯荷载-挠度全曲线数据处理软件'};

            % Create EditField_15Label_3
            app.EditField_15Label_3 = uilabel(app.UIFigure);
            app.EditField_15Label_3.HorizontalAlignment = 'right';
            app.EditField_15Label_3.Position = [287 681 76 22];
            app.EditField_15Label_3.Text = '(Mpa)';

            % Create EditField_15Label_4
            app.EditField_15Label_4 = uilabel(app.UIFigure);
            app.EditField_15Label_4.HorizontalAlignment = 'right';
            app.EditField_15Label_4.Position = [327 649 35 22];
            app.EditField_15Label_4.Text = '(Mpa)';

            % Create EditField_15Label_5
            app.EditField_15Label_5 = uilabel(app.UIFigure);
            app.EditField_15Label_5.HorizontalAlignment = 'right';
            app.EditField_15Label_5.Position = [329 617 35 22];
            app.EditField_15Label_5.Text = '(Mpa)';

            % Create Label
            app.Label = uilabel(app.UIFigure);
            app.Label.HorizontalAlignment = 'right';
            app.Label.Position = [138 681 84 22];
            app.Label.Text = '数据1峰值荷载';

            % Create EditField
            app.EditField = uieditfield(app.UIFigure, 'text');
            app.EditField.ValueChangedFcn = createCallbackFcn(app, @EditFieldValueChanged, true);
            app.EditField.Position = [230 681 86 22];

            % Create Label_2
            app.Label_2 = uilabel(app.UIFigure);
            app.Label_2.HorizontalAlignment = 'right';
            app.Label_2.Position = [138 649 84 22];
            app.Label_2.Text = '数据2峰值荷载';

            % Create EditField_2
            app.EditField_2 = uieditfield(app.UIFigure, 'text');
            app.EditField_2.Position = [230 649 86 22];

            % Create Label_3
            app.Label_3 = uilabel(app.UIFigure);
            app.Label_3.HorizontalAlignment = 'right';
            app.Label_3.Position = [138 617 84 22];
            app.Label_3.Text = '数据3峰值荷载';

            % Create EditField_3
            app.EditField_3 = uieditfield(app.UIFigure, 'text');
            app.EditField_3.Position = [230 617 86 22];

            % Create Label_7
            app.Label_7 = uilabel(app.UIFigure);
            app.Label_7.HorizontalAlignment = 'right';
            app.Label_7.Position = [365 681 146 22];
            app.Label_7.Text = '数据1峰值荷载对应挠度';

            % Create EditField_7
            app.EditField_7 = uieditfield(app.UIFigure, 'text');
            app.EditField_7.Position = [519 681 78 22];

            % Create Label_10
            app.Label_10 = uilabel(app.UIFigure);
            app.Label_10.HorizontalAlignment = 'right';
            app.Label_10.Position = [366 649 146 22];
            app.Label_10.Text = '数据2峰值荷载对应挠度';

            % Create EditField_8
            app.EditField_8 = uieditfield(app.UIFigure, 'text');
            app.EditField_8.Position = [520 649 77 22];

            % Create Label_11
            app.Label_11 = uilabel(app.UIFigure);
            app.Label_11.HorizontalAlignment = 'right';
            app.Label_11.Position = [367 617 146 22];
            app.Label_11.Text = '数据3峰值荷载对应挠度';

            % Create EditField_9
            app.EditField_9 = uieditfield(app.UIFigure, 'text');
            app.EditField_9.Position = [521 617 76 22];

            % Create Label_13
            app.Label_13 = uilabel(app.UIFigure);
            app.Label_13.HorizontalAlignment = 'right';
            app.Label_13.Position = [370 585 146 22];
            app.Label_13.Text = '峰值荷载对应挠度均值';

            % Create Label_8
            app.Label_8 = uilabel(app.UIFigure);
            app.Label_8.HorizontalAlignment = 'right';
            app.Label_8.Position = [130 585 84 22];
            app.Label_8.Text = '峰值应力均值';

            % Create EditField_13
            app.EditField_13 = uieditfield(app.UIFigure, 'text');
            app.EditField_13.Position = [230 585 86 22];

            % Create EditField_14
            app.EditField_14 = uieditfield(app.UIFigure, 'text');
            app.EditField_14.Position = [524 585 73 22];

            % Create Label_9
            app.Label_9 = uilabel(app.UIFigure);
            app.Label_9.HorizontalAlignment = 'right';
            app.Label_9.Position = [139 544 84 18];
            app.Label_9.Text = '第';

            % Create EditField_15
            app.EditField_15 = uieditfield(app.UIFigure, 'text');
            app.EditField_15.ValueChangedFcn = createCallbackFcn(app, @EditField_15ValueChanged, true);
            app.EditField_15.Position = [231 544 86 18];

            % Create EditField_15Label_15
            app.EditField_15Label_15 = uilabel(app.UIFigure);
            app.EditField_15Label_15.HorizontalAlignment = 'right';
            app.EditField_15Label_15.Position = [328 585 35 22];
            app.EditField_15Label_15.Text = '(Mpa)';

            % Create UITable
            app.UITable = uitable(app.UIFigure);
            app.UITable.ColumnName = {'挠度'; '荷载'};
            app.UITable.RowName = {};
            app.UITable.SelectionType = 'column';
            app.UITable.CellEditCallback = createCallbackFcn(app, @UITableCellEdit, true);
            app.UITable.DisplayDataChangedFcn = createCallbackFcn(app, @UITableDisplayDataChanged, true);
            app.UITable.Position = [674 55 257 595];

            % Create EditField_15Label_17
            app.EditField_15Label_17 = uilabel(app.UIFigure);
            app.EditField_15Label_17.BackgroundColor = [0 0.4471 0.7412];
            app.EditField_15Label_17.HorizontalAlignment = 'center';
            app.EditField_15Label_17.FontColor = [1 1 1];
            app.EditField_15Label_17.Position = [674 673 139 23];
            app.EditField_15Label_17.Text = '平均后的实验曲线数据';

            % Create Button_4
            app.Button_4 = uibutton(app.UIFigure, 'state');
            app.Button_4.ValueChangedFcn = createCallbackFcn(app, @Button_4ValueChanged, true);
            app.Button_4.Text = '生成';
            app.Button_4.FontSize = 8;
            app.Button_4.Position = [821 675 53 19];

            % Create Button_5
            app.Button_5 = uibutton(app.UIFigure, 'state');
            app.Button_5.ValueChangedFcn = createCallbackFcn(app, @Button_5ValueChanged2, true);
            app.Button_5.Text = '导出';
            app.Button_5.FontSize = 8;
            app.Button_5.Position = [878 672 54 22];

            % Create EditField_15Label_18
            app.EditField_15Label_18 = uilabel(app.UIFigure);
            app.EditField_15Label_18.HorizontalAlignment = 'right';
            app.EditField_15Label_18.Position = [596 681 31 22];
            app.EditField_15Label_18.Text = '(mm)';

            % Create EditField_15Label_19
            app.EditField_15Label_19 = uilabel(app.UIFigure);
            app.EditField_15Label_19.HorizontalAlignment = 'right';
            app.EditField_15Label_19.Position = [596 651 31 22];
            app.EditField_15Label_19.Text = '(mm)';

            % Create EditField_15Label_20
            app.EditField_15Label_20 = uilabel(app.UIFigure);
            app.EditField_15Label_20.HorizontalAlignment = 'right';
            app.EditField_15Label_20.Position = [596 617 31 22];
            app.EditField_15Label_20.Text = '(mm)';

            % Create EditField_15Label_21
            app.EditField_15Label_21 = uilabel(app.UIFigure);
            app.EditField_15Label_21.HorizontalAlignment = 'right';
            app.EditField_15Label_21.Position = [596 585 31 22];
            app.EditField_15Label_21.Text = '(mm)';

            % Show the figure after all components are created
            app.UIFigure.Visible = 'on';
        end
    end

    % App creation and deletion
    methods (Access = public)

        % Construct app
        function app = KangWan

            % Create UIFigure and components
            createComponents(app)

            % Register the app with App Designer
            registerApp(app, app.UIFigure)

            if nargout == 0
                clear app
            end
        end

        % Code that executes before app deletion
        function delete(app)

            % Delete UIFigure when app is deleted
            delete(app.UIFigure)
        end
    end
end
怎么
