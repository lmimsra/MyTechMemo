# scalaでplay2.5.Xを使って開発してみたメモ  

- play-Bootstrapのテキスト  
> 1.1-P25-B4	play-bootstrap	Play 2.5.6	Bootstrap 4.0.0-alpha	jQuery 2.2.3  

http://adrianhurt.github.io/play-bootstrap/1.1-P25-B4/docs/

- DECODE(crypt_str,pass_str)  
暗号化された文字列 crypt_str の復号化を、pass_str をパスワードとして使用して行います。crypt_str は、ENCODE() から返された文字列であるべきです。  

- ENCODE(str,pass_str)
pass_str を使用し、str をパスワードとして暗号化します。結果を復号化するには DECODE() を用います。  
結果は、str と同じ長さのバイナリ列になります。  
暗号化の強度は、ランダム発生器の質によります。短い文字列でも十分です。  
