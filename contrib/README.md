# contrib — Fedora セットアップ一式 / Fedora setup files

新しいマシンでこのフォークの指紋ロック解除環境を再現するための一式。
背景・トラブルシュートの詳細は linux-config リポジトリの `sway/README.md` §6 を参照。

- `fprintd-resume.service` — サスペンド復帰時に fprintd を再起動して古い device claim を掃除する oneshot（`/etc/systemd/system/` へ）
- `swaylock.pam` — swaylock 用 PAM 設定。**password-auth のみ**が正解（指紋はこのフォークが D-Bus で直接処理するため、ここに `pam_fprintd` を足すと二重プロンプトで干渉する）（`/etc/pam.d/swaylock` へ）

## 新規マシンでのセットアップ（Fedora）

```sh
# 1. 依存パッケージ
sudo dnf install -y meson ninja-build gcc pkgconf-pkg-config \
  wayland-devel wayland-protocols-devel libxkbcommon-devel \
  cairo-devel gdk-pixbuf2-devel pam-devel glib2-devel scdoc \
  fprintd fprintd-pam

# 2. ビルドとインストール（/usr/local/bin/swaylock が /usr/bin/swaylock を PATH 優先で差し替え）
#    fprintd ブランチに全修正済み: resume 後の re-claim 修正、D-Bus XML の vendor 化
git clone -b fprintd git@github.com:kter/swaylock-fprintd.git ~/workspace/swaylock-fprintd
cd ~/workspace/swaylock-fprintd
meson setup build --prefix=/usr/local -Dpam=enabled -Dgdk-pixbuf=enabled -Dman-pages=enabled
ninja -C build
sudo ninja -C build install

# 3. システム設定の配置
sudo cp contrib/swaylock.pam /etc/pam.d/swaylock
sudo cp contrib/fprintd-resume.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable fprintd-resume.service

# 4. 指紋の登録（ハードウェアごとに必須 — 旧マシンからは移行できない）
fprintd-enroll

# 5. sway 側: swaylock 呼び出しに -p（--fingerprint）を付ける
#    （linux-config リポジトリの sway/config では設定済み）
```

## 動作確認

- ロック（`swaylock -f -p -c 000000`）→ 指紋で解除できること
- サスペンド → 復帰 → 指紋で解除できること。
  `journalctl -b | grep -iE 'fprintd|claimed'` に
  `vanished while claimed; dropping stale device` が出て再 claim していれば正常
