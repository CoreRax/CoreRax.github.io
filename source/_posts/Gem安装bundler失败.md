title: Gem安装bundler失败的解决方案
---

安装好ruby之后继续安装bundler，输入：gem install bundler；
出现错误：
ERROR: While executing gem ... (Gem::RemoteFetcher::FetchError)
Errno::ECONNRESET: An existing connection was forcibly closed by the remote
host. - SSL_connect (https://api.rubygems.org/gems/bundler-1.10.6.gem)
原因分析：
gem源里面的连接用的是https，但是实际我们发起的却是http，被服务器终止掉了
解决办法：
gem source -r  https://rubygems.org/

删除安全连接

gem source -a  http://rubygems.org/
添加非https连接


或者使用淘宝源？