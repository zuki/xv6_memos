# `sd.c`の関数

## 公開関数

- void sd_init();
- void sdrw(struct buf *b);
- void sd_intr();

## プライベート関数

- static void sd_start(struct buf *b);
- static void sd_delayus(uint32_t cnt);
- static int sdInit();
- static int sdReadSCR();
- static void sdParseCID();
- static void sdParseCSD();
- static int sdSendCommandP(EMMCCommand* cmd, int arg);
- static int sdAppSendOpCond(int arg);
- static int sdSendCommand(int index);
- static int sdSendCommandA(int index, int arg);
- static int sdWaitForInterrupt(unsigned int mask);
- static int sdWaitForCommand():
- static int sdWaitForData();
- static uint32_t sdGetClockDivider(uint32_t freq):
- static int sdSetClock(int freq);
- static int sdResetCard(int resetType);
- static void sdInitGPIO();
- int sdGetBaseClock();

## 構造体

- コマンド用

    ```
    typedef struct EMMCCommand
    {
        const char* name;
        unsigned int code;
        unsigned char resp;
        unsigned char rca;
        int delay;
    } EMMCCommand;
    ```

- `SDカードディスクリプタ用

    ```
    typedef struct SDDescriptor {
        // Static information about the SD Card.
        unsigned long long capacity;
        unsigned int cid[4];
        unsigned int csd[4];
        unsigned int scr[2];
        unsigned int ocr;
        unsigned int support;
        unsigned int fileFormat;
        unsigned char type;
        unsigned char uhsi;
        unsigned char init;
        unsigned char absent;

        // Dynamic information.
        unsigned int rca;
        unsigned int cardState;
        unsigned int status;

        EMMCCommand* lastCmd;
        unsigned int lastArg;
    } SDDescriptor;
    ```

## コマンド表

```
static EMMCCommand sdCommandTable[] =
  {
    { "GO_IDLE_STATE", 0x00000000|CMD_RSPNS_NO                             , RESP_NO , RCA_NO  ,0},
    { "ALL_SEND_CID" , 0x02000000|CMD_RSPNS_136                            , RESP_R2I, RCA_NO  ,0},
    { "SEND_REL_ADDR", 0x03000000|CMD_RSPNS_48                             , RESP_R6 , RCA_NO  ,0},
    { "SET_DSR"      , 0x04000000|CMD_RSPNS_NO                             , RESP_NO , RCA_NO  ,0},
    { "SWITCH_FUNC"  , 0x06000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "CARD_SELECT"  , 0x07000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_YES ,0},
    { "SEND_IF_COND" , 0x08000000|CMD_RSPNS_48                             , RESP_R7 , RCA_NO  ,100},
    { "SEND_CSD"     , 0x09000000|CMD_RSPNS_136                            , RESP_R2S, RCA_YES ,0},
    { "SEND_CID"     , 0x0A000000|CMD_RSPNS_136                            , RESP_R2I, RCA_YES ,0},
    { "VOLT_SWITCH"  , 0x0B000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "STOP_TRANS"   , 0x0C000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_NO  ,0},
    { "SEND_STATUS"  , 0x0D000000|CMD_RSPNS_48                             , RESP_R1 , RCA_YES ,0},
    { "GO_INACTIVE"  , 0x0F000000|CMD_RSPNS_NO                             , RESP_NO , RCA_YES ,0},
    { "SET_BLOCKLEN" , 0x10000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "READ_SINGLE"  , 0x11000000|CMD_RSPNS_48 |CMD_IS_DATA  |TM_DAT_DIR_CH, RESP_R1 , RCA_NO  ,0},
    { "READ_MULTI"   , 0x12000000|CMD_RSPNS_48 |TM_MULTI_DATA|TM_DAT_DIR_CH, RESP_R1 , RCA_NO  ,0},
    { "SEND_TUNING"  , 0x13000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SPEED_CLASS"  , 0x14000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_NO  ,0},
    { "SET_BLOCKCNT" , 0x17000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "WRITE_SINGLE" , 0x18000000|CMD_RSPNS_48 |CMD_IS_DATA  |TM_DAT_DIR_HC, RESP_R1 , RCA_NO  ,0},
    { "WRITE_MULTI"  , 0x19000000|CMD_RSPNS_48 |TM_MULTI_DATA|TM_DAT_DIR_HC, RESP_R1 , RCA_NO  ,0},
    { "PROGRAM_CSD"  , 0x1B000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SET_WRITE_PR" , 0x1C000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_NO  ,0},
    { "CLR_WRITE_PR" , 0x1D000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_NO  ,0},
    { "SND_WRITE_PR" , 0x1E000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "ERASE_WR_ST"  , 0x20000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "ERASE_WR_END" , 0x21000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "ERASE"        , 0x26000000|CMD_RSPNS_48B                            , RESP_R1b, RCA_NO  ,0},
    { "LOCK_UNLOCK"  , 0x2A000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "APP_CMD"      , 0x37000000|CMD_RSPNS_NO                             , RESP_NO , RCA_NO  ,100},
    { "APP_CMD"      , 0x37000000|CMD_RSPNS_48                             , RESP_R1 , RCA_YES ,0},
    { "GEN_CMD"      , 0x38000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},

    // APP commands must be prefixed by an APP_CMD.
    { "SET_BUS_WIDTH", 0x06000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SD_STATUS"    , 0x0D000000|CMD_RSPNS_48                             , RESP_R1 , RCA_YES ,0}, // RCA???
    { "SEND_NUM_WRBL", 0x16000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SEND_NUM_ERS" , 0x17000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SD_SENDOPCOND", 0x29000000|CMD_RSPNS_48                             , RESP_R3 , RCA_NO  ,1000},
    { "SET_CLR_DET"  , 0x2A000000|CMD_RSPNS_48                             , RESP_R1 , RCA_NO  ,0},
    { "SEND_SCR"     , 0x33000000|CMD_RSPNS_48|CMD_IS_DATA|TM_DAT_DIR_CH   , RESP_R1 , RCA_NO  ,0},
  };
```
