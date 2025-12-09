---
title: 「工具」mac设置ssh不掉线
date: 2023-07-27 20:28:15
tags: [工具]
---
我们在mac下面用终端连接远程服务器, 如果长时间不操作, 会发现连接会断开, 所以不得不重新连接, 很多时候我们不希望出现这种情况, 这个时候可以在服务器或者客户端设置  
mac客户端设置  
  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line"># 打开</span><br><span class="line">vi ~/.ssh/config</span><br><span class="line"># 添加, 60s向服务端请求一次</span><br><span class="line">ServerAliveInterval = 60</span><br><span class="line"># 设置文件权限</span><br><span class="line">chmod 600 ~/.ssh/config</span><br></pre></td></tr></tbody></table>

> 切记, 权限一定要设置否则没有效果

Linux服务器设置  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><span class="line"># 打开(服务端是文件 sshd_config)</span><br><span class="line">vi /etc/ssh/sshd_config</span><br><span class="line"># 添加</span><br><span class="line">ClientAliveInterval 60</span><br><span class="line">ClientAliveCountMax 1</span><br></pre></td></tr></tbody></table>

> SSH Server 每 60 秒就会自动发送一个信号给 Client，而等待 Client 回应