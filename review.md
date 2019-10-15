# 长密钥ID碰撞

[The Long Key ID Collider](http://nullprogram.com/blog/2019/07/22/)

在过去几周里，我花了很多时间在OpenPGP秘钥上。这是由于打磨我的密码导出PGP秘钥生成工具导致的。我加深了解了它的内部实现，它让我探索了格式化的各个方面，尝试有趣的东西，然后评价各种各样的OpenPGP实现应答到怪异的输入中。

为了一个特别酷的技巧，看一看这两个我昨天生成的私钥。这个是第一个：

```linux
-----BEGIN PGP PRIVATE KEY BLOCK-----

xVgEXTU3gxYJKwYBBAHaRw8BAQdAjJgvdh3N2pegXPEuMe25nJ3gI7k8gEgQvCor
AExppm4AAQC0TNsuIRHkxaGjLNN6hQowRMxLXAMrkZfMcp1DTG8GBg1TzQ9udWxs
cHJvZ3JhbS5jb23CXgQTFggAEAUCXTU3gwkQmSpe7h0QSfoAAGq0APwOtCFVCxpv
d/gzKUg0SkdygmriV1UmrQ+KYx9dhzC6xwEAqwDGsSgSbCqPdkwqi/tOn+MwZ5N9
jYxy48PZGZ2V3ws=
=bBGR
-----END PGP PRIVATE KEY BLOCK-----
```

这是第二个：

```linux
-----BEGIN PGP PRIVATE KEY BLOCK-----

xVgEXTU3gxYJKwYBBAHaRw8BAQdAzjSPKjpOuJoLP6G0z7pptx4sBNiqmgEI0xiH
Z4Xb16kAAP0Qyon06UB2/gOeV/KjAjCi91MeoUd7lsA5yn82RR5bOxAkzQ9udWxs
cHJvZ3JhbS5jb23CXgQTFggAEAUCXTU3gwkQmSpe7h0QSfoAAEv4AQDLRqx10v3M
bwVnJ8BDASAOzrPw+Rz1tKbjG9r45iE7NQEAhm9QVtFd8SN337kIWcq8wXA6j1tY
+UeEsjg+SHzkqA4=
=QnLn
-----END PGP PRIVATE KEY BLOCK-----
```

连接它们然后导入到GnuPG看一下。为了避免丢掉真实的秘钥环，尤其是私钥，使用--homedir选项来设置临时的密钥环。我将会在示例中使用这些选项。

```linux
$ gpg --import < keys.asc
gpg: key 992A5EEE1D1049FA: public key "nullprogram.com" imported
gpg: key 992A5EEE1D1049FA: secret key imported
gpg: key 992A5EEE1D1049FA: public key "nullprogram.com" imported
gpg: key 992A5EEE1D1049FA: secret key imported
gpg: Total number processed: 2
gpg:               imported: 2
gpg:       secret keys read: 2
gpg:   secret keys imported: 2
```

自从我做了这些，用户ID就是"nullprogram.com"，这些就是我的信用。"992A5EEE1D1049FA"就称作长键ID:一个64位的数值来代替键。它是全密钥ID的最低64位，160位SHA-1哈希。过去，所有人都使用短的密钥ID来标识密钥，这是密钥最低的32位。对于上面的秘钥来说，就是"1D1049FA"。然而，这被视作太短了，每个人都转向长的密钥ID，甚至整个160位的密钥ID。

密钥ID没有比密钥创建时间的SHA1哈希多处任何东西--无符号的32位unix时间戳--对于公钥是必须的。所以密钥和相关联的公钥都有同样的密钥ID。既然它们是一对密钥并且连在一起也是理所应当的。

仔细看你就会发现两个键对都有相同的密钥ID。如果你还没有从这篇文章的标题猜出来，这里有两个不同的键却又相同的密钥ID。换一种说法，我创建了一个碰撞的密钥ID。自从有这么重要GnuPG的--list-keys命令打印了整个密钥ID：

```linux
$ gpg --list-keys
---------------------
pub   ed25519 2019-07-22 [SCA]
      A422F8B0E1BF89802521ECB2992A5EEE1D1049FA
uid           [ unknown] nullprogram.com

pub   ed25519 2019-07-22 [SCA]
      F43BC80C4FC2603904E7BE02992A5EEE1D1049FA
uid           [ unknown] nullprogram.com
```

我仅仅把最低的64位作为目标，但是实际上偶然碰撞了最低68位。因此长密钥ID仍然不能足够信任的确定任何密钥。

当然这不是新闻。我也不是第一个创建长密码冲突的人。在2013年，[David Leon Gil published a long key ID collision for two 4096-bit RSA public keys](https://mailarchive.ietf.org/arch/msg/openpgp/Al8DzxTH2KT7vtFAgZ1q17Nub_g)。然而，这只是我找到的另外一个例子。他没有包含私钥也没有详细描述他是如何做到的。我知道他生成的钥匙是可行的，而不仅仅是公钥部分的垃圾，因为它们都是自签约。

创建那些密钥是比起我期望的复杂的多，有一个老旧的、聪明的办法可以让它工作。在我为passphrase2pgp所做的工作的基础上，我创建了一个单独的工具，可以创建长的密钥ID并打印两个密钥对到标准输出。

- <https://github.com/skeeto/pgpcollider>

示例：

```linux
$ go get -u github.com/skeeto/pgpcollider
$ pgpcollider --verbose > keys.asc
```

这样运行会耗费一天时间。这个工具可以选择性的协调很多机器--见--server/-S和--client/-C选项--一起工作大大缩短总的时间。在单机上创建上面的密钥大概需要4个小时，在这个过程中大概产生10亿个密钥。如下面讨论的，我觉得很幸运的是只消耗了10亿毫秒。如果你修改程序进行简短的密钥ID碰撞，只需要几秒钟。

## 生日攻击

一个特定的细节是这种技术并不针对任何特定的密钥ID。克隆其他人的长密钥ID依旧非常耗费精力。不，这是一个[生日攻击](https://en.wikipedia.org/wiki/Birthday_attack)。从一个2^64的空间找一个碰撞，平均我仅仅需要生成2^32的例子--那个空间的平方根。在普通台式计算机上完全可行。去碰撞长密钥ID，我仅仅需要生成大概40亿ID，并在那个设置上有效率的进行成员测试。

那个最后一步说起来比做起来容易。自然地，看起来像这样(pseudo-code):

```go
seen := map of long key IDs to keys
loop forever {
    key := generateKey()
    longID := key.ID[12:20]
    if longID in seen {
        output seen[longID]
        output key
        break
    } else {
        seen[longID] = key
    }
}
```

考虑到map的大小。每一个长ID是8字节，我们期望存储2^32个。最少也需要32GB的存储才能放下所有的长ID。map本身还有一些开销。由于这些实际上是随机的查找，这一切都需要在内存中要不然其他查找方式将会非常缓慢和不切实际。

并且我还没有计算密钥的数目。作为可取之处，它们都是Ed25519密钥，因此私钥和公钥都是32字节，如果我需要就应该创建自签名。(自身签名将会比加密键大很多)。那是大约256GB的存储空间，虽然这至少可以存放在硬盘上。然而，从map中取地址我将需要至少38位，在加一些更多的数据以防止溢出。再加8字节好了。

因此，至少是64GB内存加上256GB其他存储。没什么是理想的，我们需要更多的东西。这一切依然可行，但是需要昂贵的硬件资源。我们可以做的更好。

## 种子选手中的密钥

你能注意到的是，我们可以通过更聪明的生成密钥来舍弃256GB的存储。由于我们实际上不关心密钥的安全，我们可以从一个比密钥小的种子生成密钥。不要用8字节来引用存储中的密钥，只用这8字节来存储用于制作密钥的种子。

```go
counter := rand64()
seen := map of long key IDs to 64-bit seeds
loop forever {
    seed := counter
    counter++
    key := generateKey(seed)
    longID := key.ID[12:20]
    if longID in seen {
        output generateKey(seen[longID])
        output key
        break
    } else {
        seen[longID] = seed
    }
}
```

我正在增加一个计数器来生成种子，因为我不想经历生日悖论来应用于我的种子。每一个都必须唯一。我在伪随机数上使用了SplitMix64，所以一个简单的正常生成种子是不错的。

归根到底，这仍旧使用了过多的内存。如果我们能把map的内存从64GB降低到几MB上，这不是很疯狂吗？好吧，我们可以！

## 彩虹表(Rainbow tables)

数十年来，解密高手面临一个相同的问题。它们想预先计算数十亿流行密码的hash，以便他们在日后能有效的逆转这些密码。然而，存储所有这些hash会非常的昂贵也没有必要，甚至不可行。

所以他们不会。他们用彩虹表来代替。密码hash链成一起放到一个hash链，密码hash会导致新的密码，以此类推。因此仅仅存储每个链的开始和结束。

从彩虹表中查找一个hash，从目标hash开始运行hash链算法，为每一个hash，检查是否能匹配链条的末尾。如果是这样，重新计算那个链然后注意在目标散列值之前的步骤。这就是密码。

例如，猜想密码"foo"的hash是9bfe98eb，我们有一个精简函数可以将hash映射到某个密码。在这种情况下，它将9bfe98eb映射到"bar"。一个琐碎的精简函数可能只是一个索引到一个密码列表。一个hash链从"foo"开始可能像这样：

```linux
foo -> 9bfe98eb -> bar -> 27af0841 -> baz -> d9d4bbcb
```

实际上链会更长。另一个以"apple"开始的就像这样：

```linux
apple -> 7bbc06bc -> candle -> 82a46a63 -> dog -> 98c85d0a
```

我们仅仅保存了元祖(foo, d9d4bbcb)和(apple, 98c85d0a)到我们的数据库中。如果这个链有一百万hash的长度，我们仍然只存储那两个元祖。那简直是1:1000000的压缩比例。

随后我们将面对反转hash27af0841，这个没有直接列在数据库中。因此我从这个hash开始跑整个链，直到到达最大长度(密码不存在)，或者我们识别出一个hash。

```linux
27af0841 -> baz -> d9d4bbcb
```

hash值d9d4bbcb就是列出的"foo"的hash链。所以我重新生成hash链来发现"bar"通向27af0841。密码破解了！

## 彩虹表碰撞机

我的碰撞及工作非常相似。一个hash链就像这样：由一个64位的种子开始，生成一个密钥，得到长密钥ID，使用长密钥ID作为下一个密钥的种子。

<img src="./img/collider-chain-1.png" width="100%" >

这里有很大的不同。在彩虹表中的目的是通过看链中的前一步来反向运行哈希函数。对于碰撞机，我想知道哈希链是否有冲突。只要每个链条从唯一的种子开始，就意味着我们找到了两个不同的种子，它们的长密钥ID是一样的。

或者，可能是两个不同的种子导致了相同的密钥，这不会有用，但那只是规避而已。

一个简单而有效的方法来检查两个链条是否包含相同的序列，就是按这个序列在同一个位置停止它们。它们没有在一些固定的步骤运行哈希链，而是在到达一个区别点时停止。在我的冲突中，一个区别点是长密钥ID以至少N个0比特结尾，N决定平均链长。我选择了17位。

```go
func computeChain(seed) {
    loop forever {
        key := generateKey(seed)
        longID := key.ID[12:20]
        if distinguished(longID) {
            return longID
        }
        seed = longID
    }
}
```

如果两个不同的哈希链在同一个区别点结束，它们肯定会在中间某个地方发生碰撞。

<img src="./img/collider-chain-1.png" width="100%" >

判断两个链哪里碰撞，重新生成每个链然后查找共有的首个长密钥ID。前面的步骤就是碰撞键。

```go
counter := rand64()
seen := map of long key IDs to 64-bit seeds
loop forever {
    seed := counter
    counter++
    longID := computeChain(seed)
    if longID in seen {
        output findCollision(seed, seen[longID])
        break
    } else {
        seen[longID] = seed
    }
}
```

哈希链计算的并行性令人尴尬，因此可以利用CPU的各个核心。通过这些类彩虹表，我的工具可以在几MB内存上生成和解密。额外的计算成本是生成更多链所需的时间，但不是必须的。
