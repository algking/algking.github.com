---
layout: post
category: "lisp"
description: "emacs  图片显示支持 retina"
---

###  markdown-mode org-mode下实现截图，即时显示 ###
在 mac 下主要是利用 shell 的命令 `screenshot` 实现，实际上 cmd-shift-2的
截图也是用这个命令去实现的。

#### org-mode-screenshot ####

1. `call-process-shell-command` 命令参数拼接实现截图
2. `(insert (concat "[[~/note/images/" filename ".png" "]]"))` 插入到 buffer
3. `(org-display-inline-images)` 即时显示图片


``` emacs-lisp
(defun my-org-screenshot ()
  "Take a screenshot into a time stamped unique-named file in the
same directory as the org-buffer and insert a link to this file."
  (interactive)
  (setq filename
        (concat
         (make-temp-name
          (concat (file-name-base)
                  "_"
                  (format-time-string "%Y%m%d_%H%M%S_")))
         ))
  (setq filename_real (concat "~/note/images/" filename ".png"))
  
  (call-process-shell-command "screencapture" nil nil nil
                              (concat " -i " filename_real))
  
  ;; (call-process-shell-command "convert" nil nil nil
  ;;                             (concat " -scale 50% -quality 100% " filename_2x " " filename_real ))

  (insert (concat "[[~/note/images/" filename ".png" "]]"))
  (org-display-inline-images))
```

接下来的问题是因为使用的是带 retina 的 macbook 所以图片显示会非常大，因为分辨率的问题，而 retina 实际上是利用4个像素代替一个像素的原理是实现的，所以retina dpi 会很高。

解决方案：
根据 [emacs-mac-port](https://github.com/railwaycat/emacs-mac-port/blob/7be6d9e8dbd122b060b058c0a345091debc52975/doc/emacs/macport.texi#L401 ) 
同一个目录下 存在filename filename@2x filename@2x.width=filename.width 的话，emacs-mac-port会自动支持 retina
显示，这个貌似是网站用来支持 retina 显示的 workaround。


4. 第一个方案是让截图的时候，同时生成两张图片

``` emacs-lisp
(setq filename_2x (concat "~/note/images/"  filename "@2x.png"))
(call-process-shell-command "convert" nil nil nil
    (concat " -scale 50% -quality 100% " filename_2x " " filename_real ))
    
```


 不过我在想 emacs-mac-port 应该在 c 代码层支持了 rentina 显示的方案，
 这个必须有两张图片的规则在网站可以，但是emacs 使用的如果是 retina 
 屏幕那么他们肯定是想默认就显示为 retina 模式的，于是不甘心的我就不断
 的 `grep` `occour-mode` 找到了emacs实现图片显示的核心代码的位置
 [ _emacs-mac-port/src/image.c_](https://github.com/railwaycat/emacs-mac-port/blob/7be6d9e8dbd122b060b058c0a345091debc52975/src/image.c ) 
 

 琢磨了几天看了 image.el,也用`edebug-defun`去调试了，看了 image.c
 image_load_image_io 貌似是 mac-port 的 patch 核心，作者自己实现了一套代码，并没有在原来的基础上大改.(学习~)

``` c++
#ifdef HAVE_MACGUI
static bool
png_load (struct frame *f, struct image *img)
{
  return image_load_image_io (f, img, kUTTypePNG);
}
#endif  /* HAVE_MACGUI */

```

image.c 的大致思路是这样的

1. 读取文件到内存image， 用filename 模式或者 data 模式 
2. 生成 document 也就是 buffer 用来显示 emacs 各种文本的内存，要处理
 width height这些

3. 最后把 image 调用系统 api 输出到 frame 上面

btw emacs-lisp 层的输出文本的 api 大致是这样的,display 是内部 c 的 api

``` emacs-lisp
(add-text-properties (match-beginning 0) (match-end 0)
`(display ,(create-image file)
modification-hooks
(iimage-modification-hook)))
```

作者的实现实际上是当发现有 `filename@2x` 的时候，就会用大图片去覆盖掉 `filename`
so 要不我直接让要显示的图片就是 filename@2x，然后其他处理的时候把width heigth 
这些相应的处理一下

跟着 width heigth 的处理，还有 fun@2x 类似的函数跟踪    

### Finally ###

在对@2x 预处理的时候，稍微改了一下

``` c++
/* retina 模式  */
if(enable_retina_images)
{img->target_backing_scale = 2;}
file = mac_preprocess_image_for_2x_file (f, img, file);

```

并且增加了一个配置选项,这里也学到了不少东西
+ 需要增加 M-x customize 的时候的配置首先在 c 代码层是这样的，需要添加对应
的变量 BOOL 或者 LISP 函数等

``` c++
  DEFVAR_BOOL ("enable-retina-images", enable_retina_images,
               doc: /* Non-nil means always display image in retina mode.*/);
  enable_retina_images = 0;

```

o+ 然后在 cus-start.el 增加对应的配置
> ;;; cus-start.el --- define customization properties of builtins

``` emacs-lisp
;; image.c
(enable-retina-images mac boolean "24.1")

```

最后附上图

![image](/public/img/2016-04-08-emacs-retina-support_20160409_235242_75074HPT.png)

### Markdown-mode 的截图 screenshot ###

``` emacs-lisp
(defun my-md-screenshot ()
  "Take a screenshot into a time stamped unique-named file in the
same directory as the org-buffer and insert a link to this file."
  (interactive)
  (setq filename
        (concat
         (make-temp-name
          (concat (file-name-base)
                  "_"
                  (format-time-string "%Y%m%d_%H%M%S_")) ) ".png"))
  (call-process-shell-command "screencapture" nil nil nil
                              (concat " -i ~/Documents/algking.github.com/public/img/" filename))
  (insert (concat "![image](/public/img/" filename ")"))
  (iimage-mode-buffer iimage-mode)
  )
    
```

这里提一下的是，`(iimage-mode-buffer iimage-mode)` 这个函数实际上就是
_iimage-mode_ 实现显示 inline-image 的代码


 这里即时显示图片还需要一些配置,iimage-mdoe 是根据正则表达式去
 找到对应的图片链接格式，然后去指定目录去寻找图片再调用api 去
 图片显示

``` emacs-lisp
 (setq iimage-mode-image-search-path '(list "." "~/algking.github.com/public/img/"
                                            ))
 (add-to-list 'iimage-mode-image-regex-alist ; match: ![xxx](/xxx)
              (cons (concat "!\\[.*\\](/public/img/\\("
                            iimage-mode-image-filename-regex
                            "\\))") 1))

```

