# Checksum Tool 



---

#### **功能簡介**:

此工具可將 機種名稱 ex: "12-O99-1300" 串在檔案的開頭16個byte (沒有值自動補0x00) 

串起來之後計算檔案(Head+Content)每一個byte加起來的總合 Checksum (4個byte)

最後輸出( Head + Content + Checksum) 至原輸入檔案的目錄,並且顯示輸出檔案的Size及 Checksum在 視窗畫面

---

#### tkinter 系統檔案管理員開檔



```python
from tkinter import filedialog as fd

full_path = fd.askopenfilename()
```



存檔:https://www.geeksforgeeks.org/python-asksaveasfile-function-in-tkinter/

---

#### os.path 分離路徑,檔案名稱 

https://www.runoob.com/python/python-os-path.html

```python
import os

full_path = "C:\xxxx\yyyyy\zzzz\file.bin"

f_name  = os.path.basename(full_path) 
# 取得檔案路徑 "C:\xxxx\yyyyy\zzzz"

f_path  = os.path.dirname(full_path) 
# 取得檔案名稱 "file.bin"
    
```



---

#### os.system(cmd) 在Python中執行Terminal命令

cmd = "mkdir"

os.system(cmd)  相當於在命令提示字元輸入mkdir 命令

```python
import os

file_path = C:/Users/bill_lin/Desktop/小工具/ChecksumTool_For_12-O99-1300
# 打開 檔案總管 瀏覽[輸出檔案目錄]
try:
    path = file_path
    
    # Windows cmd 斜線要反過來才可以正常動作
    cmd = "explorer " + '\"' +path + '\"'
    cmd_2 = cmd.replace('/','\\')
	# cmd_2 =  'explorer "C:\Users\bill_lin\Desktop\小工具\ChecksumTool_For_12-O99-1300" '
    
    print(cmd_2)
    os.system(cmd_2)
except:
    print("open explorer err!!")
```



#### Source Code

```python
import tkinter as tk
from tkinter import filedialog as fd
import os


# os.path.basename(path) 取得檔案名稱
# os.path.dirname(path)  取得檔案路徑


def open_btn_onclick():

    # 使用系統內建 file Dialog 開啟 [輸入檔]
    full_path = fd.askopenfilename()
    #print(full_path)

    # 取得檔案路徑 "C:\xxxx\xxxx\xxxx"
    file_name  = os.path.basename(full_path)

    # 取得檔案名稱 "xxxx.bin"
    file_path  = os.path.dirname(full_path)

    # ======== UI顯示 ==========
    path_input.config(state=tk.NORMAL)
    f_name_input.config(state=tk.NORMAL)

    #先清掉內容
    path_input.delete(0,tk.END)
    f_name_input.delete(0,tk.END)

    path_input.insert(0,file_path)
    f_name_input.insert(0,file_name)

    path_input.config(state=tk.DISABLED)
    f_name_input.config(state=tk.DISABLED)
    # ======== UI顯示 ==========

    # 產生head 標頭16byte Device info b'12-O99-1300\x00\x00\x00\x00\x00'
    head_array_16 = bytearray(16) 
    dev_array = bytearray(dev_input.get().encode('utf-8'))

    for idx in range(0 ,len(dev_array)):
        head_array_16[idx] = dev_array[idx]

    head = bytes(head_array_16)
    #print(head)
    
    # 將 [輸入檔案] 內容放進暫存 (RAM)
    f = open(full_path,"rb")  
    content = f.read()
    f.close()

    # 將 [標頭] 和 [輸入檔案] 串成 [合併資料] 並轉換成bytearray
    head_content_arr = bytearray(head + content)
    #print(head_content_arr)

    # 計算SUM: 將[合併資料] 每一個byte資料加起來 
    sum = 0
    for idx in range(0 , len(head_content_arr)):
        byte = head_content_arr[idx]
        sum += byte
    #print(sum)
    
    # 把計算出來的[Checksum] 轉換成bytes 格式
    tail = sum.to_bytes( length=4 , byteorder="big")

    # 把 [標頭]  [輸入檔案]  [Checksum] 串成 [輸出檔案內容]
    f_output_content = head + content + tail

    # 產生 [輸出檔案路徑名稱] ex: 'C:\xxx\xxxx\add_Checksum_xxxxx.bin'
    output_file_path = file_path + "/add_Checksum_" + file_name
    print(file_path)


    # 檔案輸出
    f_out = open( output_file_path ,"wb")
    f_out.write(f_output_content) 
    f_out.close()


    # ======== UI顯示 ==========
    chks_Text.config(state=tk.NORMAL)
    o_Size_Text.config(state=tk.NORMAL)

    #先清掉內容
    chks_Text.delete(0.0,tk.END)
    o_Size_Text.delete(0.0,tk.END)

    sum_str = hex(sum).upper()
    # print('Checksum:',sum_str)
    chks_Text.insert(1.0,sum_str)

    output_size = os.path.getsize(output_file_path)
    # print('Size:',output_size)
    size_HEX = hex(output_size)
    print(size_HEX)

    o_Size_Text.insert(1.0, str(output_size) +" ("+ size_HEX + ")" )

    chks_Text.config(state=tk.DISABLED)
    o_Size_Text.config(state=tk.DISABLED)
    # ======== UI顯示 ==========

    # 打開 檔案總管 瀏覽[輸出檔案目錄]
    try:
        path = file_path
        
        cmd = "explorer " + '\"' +path + '\"'
        cmd_2 = cmd.replace('/','\\')

        print(cmd_2)
        os.system(cmd_2)
    except:
        print("open explorer err!!")

    


def Window_on_Close():
    global win

    file_name = 'dev_save.txt'
    dev_name_record = dev_input.get()
    print(dev_name_record)

    try:
        f = open(file_name,'r')        
        f.close()
        print('------------Delete old record file (X)-------')
        os.remove(file_name)
    except:
        print('-------Record file not exisit (!)--------')
    finally:        
        f = open(file_name,'w')
        f.write(dev_name_record)
        f.close()
        print('-----new record file create (^_^)-------')

    win.destroy()

win = tk.Tk()
win.title("ChecksumTool")
win.geometry("520x220")
win.resizable(1,0)
win.protocol("WM_DELETE_WINDOW", Window_on_Close)

# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
frame_1 = tk.Frame(win)

dev_label = tk.Label(frame_1,text="Device name:",font=('consolas', 13))
dev_label.pack(side=tk.LEFT,fill=tk.BOTH)

global dev_input
dev_input = tk.Entry(frame_1,width=16,font=('consolas', 13))
dev_input.pack(side=tk.LEFT,fill=tk.BOTH,padx=10)
dev_input.insert(0,"12-O99-1300")

frame_1.pack(side=tk.TOP,fill=tk.BOTH,pady=10,padx=10)
# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

frame_2 = tk.Frame(win)

fp_lbl = tk.Label(frame_2,text="Path:", font=('consolas', 13))
fp_lbl.pack(side=tk.LEFT)

global path_input
path_input = tk.Entry(frame_2,font=('consolas', 11))
path_input.pack(side=tk.LEFT,padx=5,fill=tk.X,expand=1)
path_input.config(state=tk.DISABLED)

open_btn = tk.Button(frame_2,text="open",command=open_btn_onclick , font=('consolas', 13))
open_btn.pack(side=tk.LEFT,padx=5)

frame_2.pack(side=tk.TOP,fill=tk.BOTH,pady=10,padx=10)
# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

frame_3 = tk.Frame(win)

fn_lbl = tk.Label(frame_3,text="Name:",font=('consolas', 13))
fn_lbl.pack(side=tk.LEFT)

global f_name_input
f_name_input = tk.Entry(frame_3,font=('consolas', 11),width=35)
f_name_input.pack(side=tk.LEFT, padx=5 ,fill=tk.X)
f_name_input.config(state=tk.DISABLED)

frame_3.pack(side=tk.TOP,fill=tk.BOTH,pady=10,padx=10)
# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

frame_4 = tk.Frame(win)

chks_lbl = tk.Label(frame_4,text="Checksum:" ,font=('consolas', 13))
chks_lbl.pack(side=tk.LEFT)

global chks_Text
chks_Text = tk.Text(frame_4,width=9,height=1,font=('consolas', 11))
chks_Text.pack(side=tk.LEFT,padx=5)
chks_Text.insert(1.0,"")
chks_Text.config(state=tk.DISABLED)

empty_lbl = tk.Label(frame_4)
empty_lbl.pack(side=tk.LEFT,padx=30)

o_Size_lbl = tk.Label(frame_4,text="Size:" ,font=('consolas', 13))
o_Size_lbl.pack(side=tk.LEFT)

global o_Size_Text
o_Size_Text = tk.Text(frame_4,width=16,height=1,font=('consolas', 11))
o_Size_Text.pack(side=tk.LEFT,padx=5)
o_Size_Text.insert(1.0,"")
o_Size_Text.config(state=tk.DISABLED)


frame_4.pack(side=tk.TOP,fill=tk.BOTH,pady=10,padx=10)
# xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

if __name__ == "__main__":

    file_name = 'dev_save.txt'
   
    # 確認 dev_save.txt 檔案是否存在
    try:
        f = open(file_name,'r') 
        try:       
            dev_name = f.read()
        except:
            dev_name = ''

        dev_input.delete(0,tk.END)
        dev_input.insert(0,dev_name)        
        print("read device name from record file:",dev_name)
        f.close()               
    except:
        print(file_name," not exisit ! ")
        



    win.mainloop()
```

