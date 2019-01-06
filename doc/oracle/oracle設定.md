# oracle設定
## 基礎
- dbf他:データベースファイル（下記3種のファイルで構成）  
	1. データファイル  
		- ユーザーが使用するデータ(データそのもの)
		- Oracleが管理上必要なデータ(権限設定etc)
			- [データファイルの論理構造（BEST（Block,Extent,Segment,Tablespace））](#データファイルの論理構造)
			- [表領域の種類](#表領域の種類)
	2. 制御ファイル  
		- データベースの物理構造（物理的にデータが保存されている場所）の管理(dbf,redologの位置)
	3. REDOログファイル  

</br>

- インスタンス:メモリ上に展開されたDB作業場所（下記2種で構成）
	1. SGA(System Global Erea)  
		- データベースバッファキャッシュ  
		- REDOログバッファ  
		- 共有プール（Shared Pool）  
			- ディクショナリキャッシュ  
			- ライブラリキャッシュ  
	2. バックグラウンドプロセス
		- DBWR(Data Base WRiter)
		- LGWR(LoG WRiter)
		- CKPT(ChecK PoinT):DBWRへ書き込みタイミングを指示
		- SMON(System MOniter):datafile,redologの整合チェック
		- PMON(Process MONiter):ユーザー実行プロセスの問題チェック

---
### インストール1(Normal)
参考
https://ts0818.hatenablog.com/entry/2017/12/24/212023

グローバルDB名：orcl  
パスワード：oracle 
サーバ側の設定は listener.ora  
クライアント側の設定は tnsnames.ora  
listener.ora [C:\app\oracle\product\12.2.0\dbhome_1\network\admin\listener.ora](#サーバー設定)  
tnsnames.ora [C:\app\oracle\product\12.2.0\dbhome_1\network\admin\tnsnames.ora](#クライアント設定)  
DBfiles C:\app\oracle\oradata\orcl  
SID:orcl  


#### ユーザー作成
- DB管理で使用するユーザー（インストール時に作成）
	- sys
	- system

</br>

- [DB接続](#DB接続)  
  最初はローカル接続を行い、ユーザーを作成する。  
  例：
    sqlplus /nolog  
    connect sys/oracle as sysdba  OS認証なし  
	connect / as sysdba  OS認証あり  
  そののち、そのユーザーでリモート接続する。  
</br>

- ユーザーの作成
	- 表領域の作成
		- CREATE TABLESPACE <表領域>
		- DATAFILE '<データファイル名>'
		- SIZE <サイズ>M
		完成文(data部分)
		CREATE TABLESPACE ts_cpa
		DATAFILE 'C:\app\oracle\oradata\orcl\ts_cpa.dbf'
		SIZE 100M
		;
		完成文(index部分)
		CREATE TABLESPACE index_cpa
		DATAFILE 'C:\app\oracle\oradata\orcl\index_cpa.dbf'
		SIZE 5M
		;
</br>
	- USERの作成：CPA
		- ユーザー名		：CPA
		- パスワード		：cpdev!CPA
		- デフォルト表領域		：TS_CPA
		- クォータ				：5M
		- デフォルト一時表領域	：temp
		ユーザー作成  
		CREATE USER CPA IDENTIFIED BY cpdev/CPA
		DEFAULT TABLESPACE TS_CPA
		TEMPORARY TABLESPACE temp QUOTA 5M ON TS_CPA
		;
		確認
		select username,password,default_tablespace,temporary_tablespace  
		from dba_users
		;
		select tablespace_name,username,max_bytes
		from dba_ts_quotas
		;
		ユーザー権限付与  
		grant create session to CPA  
		;  
		grant create table to CPA  
		;  
		確認


</br>


---
### インストール2(PDB)
EE2(oragable database)  
グローバルDB名：orcl2  
パスワード：oracle2  
C:\app\kantou\virtual\product\12.2.0\dbhome_1\network\admin\tnsnames.ora

Grobal dbname:  
SID:

User Setting  
system  
sys：orcl





---
# リンク
### データファイルの論理構造
- Block：oracleが認識できる最小単位。データを保存。
- Extent：連続したBlockの塊。Blockでは管理しずらいので塊にして管理。
- Segment：1つ以上のExtentの塊。テーブルや索引は論理構造とするとSegmentと同等。
- Tablespace：表領域。1つ以上のSegment（テーブルや索引）が保存される。1つ以上のデータファイルから構成される。

### 表領域の種類
- SYSTEM表領域：データディクショナリ。ユーザー・権限情報、論理構造（BEST）を保存。
- データディクショナリの種類  
	1. DBA_～：全ての情報。管理者権限保有者のみアクセス可能。  
	2. ALL_～：参照するユーザーにオブジェクト権限が与えられたtableの情報。  
	3. USER_～：参照するユーザーが保有するtableの情報。  
		- SYSAUX表領域：SYSMET表領域のAuxiliary（補助）。SYSTEM表領域に格納しないオプション機能を保存。
		- UNDO表領域：Undo Tablespace
		- 一時表領域：Tmporary Tablespace　メモリ上で作業しきらない一時作業を物理ファイルで行う。
		- ユーザー作成表領域：ユーザー管理のTablespace
			- オブジェクトの種類
				1. table（テーブル）
				2. index（索引）
				3. view（ビュー）
				4. synonym（別名）


### サーバー設定
listener.ora  
LISTENER =  
　(DESCRIPTION_LIST =  
　　(DESCRIPTION =  
　　　(ADDRESS = (PROTOCOL = TCP)(HOST = [②])(PORT = [③]))  
　　　(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))  
　　)  
　)  

SID_LIST_LISTENER =
　(SID_LIST =
　　(SID_DESC =
　　　(SID_NAME = [④])
　　　(ORACLE_HOME = C:\app\oracle\product\12.2.0\dbhome_1)
　　　(PROGRAM = extproc)
　　　(ENVS = "EXTPROC_DLLS=ONLY:C:\app\oracle\product\12.2.0\dbhome_1\bin\oraclr12.dll")
　　)
　)

詳しくは
http://otndnld.oracle.co.jp/document/products/oracle10g/102/doc_cd/network.102/B19209-01/listener.htm

### クライアント設定
tnsnames.ora　Transparent Network Substrate(透過的ネットワーク基質）  
[①] =  
　(DESCRIPTION =  
　　(ADDRESS_LIST =  
　　　　(ADDRESS = (PROTOCOL = TCP)(HOST = [②])(PORT = [③]))  
　　)  
　　(CONNECT_DATA =  
　　　　(SID = [④])  
　　)  
　)  
①任意な名前SQLPLUSなどで接続時に使用する。  
②サーバのIPアドレス（またはPC名）  
③listener.oraに記述されているポートとあわせる。  
④listener.oraに記述されているSID_NAMEにあわせる。  

以上を保存  

５、SQLPLUSを使いサーバに接続  
例）アカウント名：test パスワード：testの場合。  

ユーザ名を登録してください: test/test@[①]  

①は4のtnsnames.oraで設定した①の名前  

以上で接続されるはず。  



### DB接続    

	- sqlplus user/passwd@インスタンス  
		- 例: sqlplus system/orcl@orcl
		https://oracle-chokotto.com/ora_lsnrcomm.html

---

### OTN user
ia230028-3415@tbi.t-com.ne.jp
OTNoraclegahosii1@

---
### 参考サイト

インストール
https://thinkit.co.jp/author/10954

サンプルスキーマの投入
https://qiita.com/hobata/items/0bed0d1b2ed0566d2740

削除
https://ts0818.hatenablog.com/entry/2017/12/24/212023

サービスの削除
 （２）コマンドプロンプトで「サービス」を削除する
	次にコマンドプロンプトを使って、先程「サービス名」を確認したサービスを削除します。
	まずは、コマンドプロンプトを管理者権限で起動します。
	「スタートメニュー」＞「すべてのアプリ」＞「Windowsシステムツール」＞「コマンドプロンプト」を右クリックし「管理者として実行」をクリック

	コマンドプロンプトが起動したら次のコマンドを入力し実行します。

sc delete 〇〇〇〇

sc delete OracleJobSchedulerORCL
sc delete OracleOraDB12Home1MTSRecoveryService
sc delete OracleOraDB12Home1TNSListener
sc delete OracleServiceORCL
sc delete OracleVssWriterORCL


