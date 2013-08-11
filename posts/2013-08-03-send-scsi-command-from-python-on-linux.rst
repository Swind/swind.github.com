.. link: 
.. description: 
.. tags: Python, C/C++
.. date: 2013/08/03 10:38:06
.. title: 使用 Python 結合 C 在 Linux 發送 SCSI Command 
.. slug: send-scsi-command-from-python-on-linux

最近因為工作的需要，所以要撰寫很多 SCSI 相關的測試。
目前可以用的測試工具有 fio, iometer 以及一個自行開發的 SCSI Command 驗證的工具。
但是這些都是使用 C 撰寫的，所以要撰寫一些流程比較複雜的驗證時，就變得非常的麻煩。
絕對不是因為我想試試看 Python 或者是因為自行開發的驗證工具程式碼已經無法閱讀的原因，才想要重造輪子。

Orz 而且我受夠了手動測試的生活了 ...

所以想將測試運行的邏輯搬移到 Python 的部份，加速測試案例的開發速度。

.. TEASER_END

= 使用 C 發送 SCSI Command =

這邊第一部就是要先想辦法讓 SCSI Command 可以送到 SCSI Device 上面。
一般都是透過 ioctl 在進行，最早也有想過使用 Python 的 ioctl 來作。
但是 ioctl 的資料結構全部都定義在 .h 檔裡面，如果使用 Python 我就勢必要將這些資料結構作轉換。
所以我最後選擇第二條路，讓 Python 呼叫 C function。

.. code-block:: c

    #include <inttypes.h>
    #include <unistd.h>
    #include <fcntl.h>
    #include <stdio.h>
    #include <string.h>
    #include <errno.h>
    #include <sys/ioctl.h>
    #include <scsi/sg.h> /* take care: fetches glibc's /usr/include/scsi/sg.h */


    int execute_scsi_command(
            int fd,
            int cmd_size,   uint8_t* cmd_buffer,
            int out_size,   uint8_t* out_buffer,
            int sense_size, uint8_t* sense_buffer)
    {
        sg_io_hdr_t io_hdr;

        //Prepare SCSI Command
        memset(&io_hdr, 0, sizeof(sg_io_hdr_t));

        io_hdr.interface_id = 'S'; //required

        io_hdr.cmd_len = cmd_size; 
        io_hdr.mx_sb_len = sense_size;

        /*
         *  SG_DXFER_NONE   -   SCSI Test Unit Ready command
         *  SG_DXFER_TO_DEV -   SCSI WRITE command
         *  SG_DXFER_FROM_DEV - SCSI READ command
         */
        io_hdr.dxfer_direction = SG_DXFER_FROM_DEV;
        io_hdr.dxfer_len       = out_size;
        io_hdr.dxferp          = out_buffer; 

        io_hdr.cmdp = cmd_buffer;
        io_hdr.sbp  = sense_buffer;
        io_hdr.timeout = 20000; // 20000 millisecs = 20 seconds
        /* io_hdr.flags = 0; */     /* take defaults: indirect IO, etc */
        /* io_hdr.pack_id = 0; */
        /* io_hdr.usr_ptr = NULL; */

        if (ioctl(fd, SG_IO, &io_hdr) < 0) {
            perror("sg_simple0: Inquiry SG_IO ioctl error");
            return 0;
        }

        return io_hdr.resid;
    }

這份程式碼是從 `The Linux SCSI Generic (sg) HOWTO`_ 的 Program Example 改過來的，
透過 SG_IO 這個 ioctl 將 CDB、Data Buffer 與 Sense Buffer 傳入，等執行完畢之後我們就可以從這些 Buffer 取得 SCSI Command 結果。

接著就是編譯一個 shared library

.. code-block:: makefile

    CFLAG = -shared -fPIC -Wall
    LIB_NAME = linux_ioctl.so

    all:make_lib

    make_lib:linux_ioctl.c
            $(CC) -o $(LIB_NAME) $(CFLAG) linux_ioctl.c

    clean:
            rm -f *.so

然後就只要輸入
    
    make

就會得到一個 linux_ioctl.so 的 shared library 了。

= 使用 Python 呼叫 C function =

這部份就是我完全不熟的地方了，因為 Python 我也很少用，
所以 Python 語法非常生澀。

不過主要是透過 ctypes 來傳送 CDB、buffer pointer 到 C function 中。

.. code-block:: python

    from ctypes import *

    def hexdump(src, length, column_length=8):
        result = []
        digits = 4 if isinstance(src, unicode) else 2
        for i in xrange(0, length, column_length):
           s = src[i:i+column_length]
           hexa = b' '.join(["%0*X" % (digits, x)  for x in s])
           text = b''.join([chr(x) if 0x20 <= x < 0x7F else b'.'  for x in s])
           result.append( b"%04X   %-*s   %s" % (i, column_length*(digits + 1), hexa , text) )
        return b'\n'.join(result)

    dyn = CDLL("./linux_ioctl.so")

    fo = open("/dev/sda", "r")
    fid = fo.fileno()

    CDB = c_byte * 6
    CDB_array = CDB()

    CDB_array[0] = 0x12
    CDB_array[1] = 0x00
    CDB_array[2] = 0x00
    CDB_array[3] = 0x00
    CDB_array[4] = 96 
    CDB_array[5] = 0x00 

    CDB_p = pointer(CDB_array)

    SENSE = c_byte * 36
    SENSE_array = SENSE()

    DATA = c_byte * 96
    DATA_array = DATA()
    DATA_p = pointer(DATA_array)

    SENSE_p = pointer(SENSE_array)

    resid = dyn.execute_scsi_command(fid, 6, CDB_p, 96, DATA_p, 36, SENSE_p)

    print hexdump(DATA_array, 96 - resid)

    fo.close()

透過 CDLL 將剛剛編譯好的 shared library 載入, c_byte 的型態建立 binary array ( 這部份我覺得應該有更好的方式可以處理 )，
否則每次要建立 binary array 就都得透過 c_bytes * length 宣告 array 型態，然後在用這個型態去建立 binary array，
再透過 pointer 取得 address。
我在這部份的理解一定是有問題的，不然這樣的作法實在很奇怪。

但是無論如何，他可以動 ............

後面的

.. code-block:: python

    CDB_array[0] = 0x12
    CDB_array[1] = 0x00
    CDB_array[2] = 0x00
    CDB_array[3] = 0x00
    CDB_array[4] = 96 
    CDB_array[5] = 0x00 

就是一個標準的 INQUIRY Command 然後希望的長度是 96 bytes，
所以這個程式你可以安心測試，基本上應該是不會把硬碟弄炸掉。

.. _The Linux SCSI Generic (sg) HOWTO: http://www.tldp.org/HOWTO/SCSI-Generic-HOWTO/pexample.html
