# RCA
Root Cause Analysis、つまり障害調査に関するトピック。

## よく使用するコマンド
| コマンド | よく使うオプション | 説明 | 参考URL |
| --- | --- | --- | --- |
| dstat | -tpcdrmngy | vmstat, iostat, netstatなどを纏めた情報を取得する。| [リンク](https://qiita.com/mon_tu/items/e0074cf7b8b34fc53d2f) |
| htop | - | Process Viewer。topコマンドより見やすい。 | [リンク](https://orebibou.com/2016/05/htop%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%A7%E8%A6%9A%E3%81%88%E3%81%A6%E3%81%8A%E3%81%8D%E3%81%9F%E3%81%84%E4%BD%BF%E3%81%84%E6%96%B911%E5%80%8B/) |
| ps | auxf | 怪しい挙動のプロセスを特定する時に使う。 | [リンク](https://eng-entrance.com/linux-command-ps) |
| netstat | -anp | 外部接続確認。 | [リンク](http://d.hatena.ne.jp/nattou_curry_2/20090818/1250611294) |
| df | -h | ディスク容量の確認。 | [リンク](https://webkaru.net/linux/df-command/) |
| lsof | -c [プロセス名] | psコマンドの後に使う事が多い。 | [リンク](https://orebibou.com/2016/04/lsof%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%A7%E8%A6%9A%E3%81%88%E3%81%A6%E3%81%8A%E3%81%8D%E3%81%9F%E3%81%84%E4%BD%BF%E3%81%84%E6%96%B99%E5%80%8B/) |
| strace | -tt -s 1024 -p [プロセスID] | システムコールのトレース。 | [リンク](http://blog.livedoor.jp/sonots/archives/18193659.html) |
| ping | -c [試行回数] | ルーティング確認、死活監視。 | - |
| traceroute | -n | 特定ホスト・アドレスへの経路確認。 | [リンク](https://orebibou.com/2015/05/linux%E3%81%AEtraceroute%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%81%A7%E8%A6%9A%E3%81%88%E3%81%A6%E3%81%8A%E3%81%8D%E3%81%9F%E3%81%84%E4%BD%BF%E3%81%84%E6%96%B98%E5%80%8B/) |
| dig | -x | 名前解決の正引き・逆引き。 | [リンク](https://qiita.com/hypermkt/items/610b5042d290348a9dfa) |