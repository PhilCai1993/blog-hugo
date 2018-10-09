+++
title = "Install Chisel on macOS Mojave"
date = 2018-10-09T17:30:25+08:00
tags = ["iOS", "Xcode"]
+++

最近刚跑路到新公司, 拿到新电脑之后装了macOS Mojave & Xcode 10之后准备看代码, 于是想用[facebook/chisel](https://github.com/facebook/chisel) 帮忙来简化一下lldb的调试. 结果用`brew install` 之后, 出现了compile error. 

看看Log就知道是Xcode 10 默认的 `New Build System`的锅😅.

顺便看了一下[issue](https://github.com/facebook/chisel/issues/249), 暂时也没解决说是`Homebrew`的锅...

还不如自给自足, 解决方法蛮简单的:

自己fork一份chisel, 然后用Xcode打开`Chisel.xcodeproj`, 选File -> Project Settings, 把Build System 改成**Legacy Build System**, 然后推到自己fork的仓库. 在仓库里选择 Release -> Draft a new release. copy一下**Source code
(tar.gz)**的地址.

在本地修改一下 `usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/chisel.rb` (这个地址可以通过 `brew edit chisel` 看到). 其中url改成自己release的压缩包地址, head 改成自己的fork的仓库地址, sha256可以先不改.

运行一下
`brew install --build-from-source /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/chisel.rb`, 会报错说校验失败并且会log出一个正确的hash, 把这个rb文件里面的sha256改成log里面的hash, 再跑一下`brew install --build-from-source /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/chisel.rb`. 

应该就可以了.

懒得看👆的可以直接把
`/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/chisel.rb` 替换成下面的然后 `brew install chisel` (没测过应该可以。。。)


```ruby
class Chisel < Formula
  desc "Collection of LLDB commands to assist debugging iOS apps"
  homepage "https://github.com/facebook/chisel"
  url "https://github.com/PhilCai1993/chisel/archive/1.8.0.1.tar.gz"
  sha256 "0b64d26dff26cfd44110512fbea154f1dcd8f7703d764f165286cedf6ac3c268"
  head "https://github.com/PhilCai1993/chisel.git"

  bottle do
    cellar :any
    sha256 "eba994a5b5d1880890bab0db0741ec7f6c65f3cc4a78957355714a84b76f2fa2" => :high_sierra
    sha256 "0cde612e49ea07f54a455161b9371f6b662b450169947103dddba66fb2debe6c" => :sierra
    sha256 "82f135ec7770a425e666086b7a5d6a31b086c10e11df42fbe849354592f26a3e" => :el_capitan
  end

  def install
    libexec.install Dir["*.py", "commands"]
    prefix.install "PATENTS"

    # == LD_DYLIB_INSTALL_NAME Explanation ==
    # This make invocation calls xcodebuild, which in turn performs ad hoc code
    # signing. Note that ad hoc code signing does not need signing identities.
    # Brew will update binaries to ensure their internal paths are usable, but
    # modifying a code signed binary will invalidate the signature. To prevent
    # broken signing, this build specifies the target install name up front,
    # in which case brew doesn't perform its modifications.
    system "make", "-C", "Chisel", "install", "PREFIX=#{lib}", \
      "LD_DYLIB_INSTALL_NAME=#{opt_prefix}/lib/Chisel.framework/Chisel"
  end

  def caveats; <<~EOS
    Add the following line to ~/.lldbinit to load chisel when Xcode launches:
      command script import #{opt_libexec}/fblldb.py
  EOS
  end

  test do
    xcode_path = `xcode-select --print-path`.strip
    lldb_rel_path = "Contents/SharedFrameworks/LLDB.framework/Resources/Python"
    ENV["PYTHONPATH"] = "#{xcode_path}/../../#{lldb_rel_path}"
    system "python", "#{libexec}/fblldb.py"
  end
end

```

最后配置一下`.lldbinit`, 常规操作.

想吐槽的是如果我知道怎么让Xcode默认都不用New Build System 就不用搞这些了... 还是自己太菜🐔

