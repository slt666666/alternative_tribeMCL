# Alternatives to TribeMCL
because TribeMCL module has been disabled...

## 1. Install mcl 
```
wget http://www.micans.org/mcl/src/mcl-14-137.tar.gz
tar -zxvf mcl-14-137.tar.gz
cd mcl-14-137
./configure --enable-blast # ここで--enable-blastが必須。必要なら --prefix=~~~ も
make
make install
```

## 2. Install local blast
blastは公式から普通にインストールでOK

## 3. blast
blastの閾値は``-evalue``で設定(ここではAdachi et al., 2019に合わせて1e-8)

format等は指定せずにそのままでOK
```
makeblastdb -in elife-49956-fig3-data2-v2.txt -dbtype prot
blastp -query elife-49956-fig3-data2-v2.txt -db elife-49956-fig3-data2-v2.txt -out elife_test.txt -evalue 1e-8
```

## 4. MCL解析 & clustering ここは2種類やり方がある。
### 4.-方法1 mclblastlineコマンドを使う。
(``mcxdeblast``, ``mcxassemble``, ``mcl``, ``clmformat``をまとめて動かすコマンド)

``--mcl-I=1.4``の所でIの指定。ここではAdachi et al., 2019に合わせて1.4

dump.~~~みたいなファイルがtribe毎の結果。
```
mclblastline --mcl-I=1.4 elife_test.txt
```

## 4.-方法2 mclblastlineコマンドの中身を1つずつ実行
出力ファイル名やオプション等を細かく決めたい時(普通は方法1で良いかと思います。)
```
mcxdeblast elife_test.txt
mcxassemble -b elife_test.txt -q -r max --map
mcl elife_test.txt.sym -scheme 2 -I 1.4 --append-log=yes -o out.elife_test.txt.I14
clmformat --dump -tab elife_test.txt.tab -lump-count 500 -imx elife_test.txt.sym -dump dump.out.elife_test.txt.I14 -icl out.elife_test.txt.I14 -dir fmt.out.elife_test.txt.I14
```

## 5. 出力結果の変換
そのままの出力ファイルだと見にくいので、tribe毎にIDを並べたテキストファイルを出力するスクリプトを書いておきます

``mcl_result=``の所に4.の出力結果を入れる

``out_list_name=``の所は最終的なテキストファイルの名前
```
mcl_result=dump.out.elife_test.txt.I14
out_list_name=list.txt
echo -n >| $out_list_name
i=1
cat $mcl_result | while read LINE ; do
  IFS="$(echo -e '\t' )"
  LINE=($LINE)
  unset IFS
  for j in `seq 0 $(( ${#LINE[@]} - 1 ))`; do
    echo "tribe$i: ${LINE[$j]}" >> $out_list_name
  done
  i=$(($i+1))
done
```
