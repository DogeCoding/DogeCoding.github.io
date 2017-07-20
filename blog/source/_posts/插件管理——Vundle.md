# 插件管理——Vundle

> 都说Vim是程序员写给自己的编辑器，其中的情结可想而知。

vim因为其庞大而强劲的插件受到无比的推崇，而插件的查找和管理便成了一个问题。

[**Vundle**](https://github.com/VundleVim/Vundle.vim#about)便是一个**Github**上为了解决这个问题的项目(致敬贡献者们)，使用步骤如下：   

* ####安装Vundle，在终端输入以下代码即可

	git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle		
> ~/.vim 来自哪?
	  
	* 在**mac**中**Vim**配置文件**`.vimrc`**在 /usr/share/vim/ 下，一般是没有权限更改的,但是这个`.vimrc`是全局配置文件，我们只要更改用户配置文件即可
		* #####怎么查看**/usr/**?
		
		 1.**显示隐藏文件**，自行百度，终端中敲入代码即可。 		 2.**Finder**下或者**桌面**上的Go，文件夹输入/usr/。  
	* **mac**下的Vim用户配置文件默认是没有的，需要我们自行创建  
		* 终端下输入  
		
		   		mkdir ~/.vim
		   		touch ~/.vimrc  
		  ~/即为用户根目录。
		    
* ####在[Vim Script](http://vim-scripts.org/vim/scripts.html)选好插件
* ####在`.vimrc`文件中加一句`Bundle plugin_name`  
	* `.vimrc`示例  
	<pre><code>set nocompatible    " be iMproved
	filetype off        " required!
	set rtp+=~/.vim/bundle/vundle/
	call vundle#rc()
	" let Vundle manage Vundle
	" required!
	Bundle 'gmarik/vundle'
	" vim-scripts repos
	Bundle 'vim-plugin-foo'
	Bundle 'vim-plugin-bar'
	filetype plugin indent on    " required!</code></pre>    
* #### 执行Vundle安装命令  
	* <code>:BundleInstall</code>  
* #### Vundle的其它命令  
	* `:BundleList`	-列举出列表中(.vimrc中)配置的所有插件
	* `:BundleInstall` -安装列表中全部插件
	* `:BundleInstall!` -更新列表中全部插件
	* `:BundleSearch foo` -查找foo插件
	* `:BundleSearch! foo` -刷新foo插件缓存
	* `:BundleClean` -清除列表中没有的插件
	* `:BundleClean!` -清除列表中没有的插件


# 学习资料  
* [Git时代的VIM不完全使用教程](http://beiyuu.com/git-vim-tutorial/)  
* [使用Vundle来管理vim的插件](http://www.jianshu.com/p/062a47c365f1)  
* [zhongcq 的VIM配置](http://www.cnblogs.com/zhongcq/p/3642794.html#toc_1.15)





