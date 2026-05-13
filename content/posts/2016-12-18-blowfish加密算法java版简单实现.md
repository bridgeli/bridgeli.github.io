---
title: Blowfish加密算法Java版简单实现
author: Bridge Li
type: post
date: 2016-12-18T12:04:43+00:00

categories:
  - Java
tags:
  - Blowfish
  - 加密

---
前几天网上突然出现流言：某东发生数据泄露12G，最终某东在一篇声明中没有否认，还算是勉强承认了吧，这件事对于一般人有什么影响、应该怎么做已经有一堆人说了，所以就不凑热闹了，咱来点对程序猿来说实际点的，说一个个人认为目前比较安全的加密算法：Blowfish。  
上代码之前，先说几点Blowfish加密算法的特点：

1. 对称加密，即加密的密钥和解密的密钥是相同的；  
2. 每次加密之后的结果是不同的（这也是老夫比较欣赏的一点）；  
3. 可逆的，和老夫之前的文章介绍的md5等摘要算法不一样，他是可逆的；  
4. 速度快，加密和解密的过程基本上由ADD和XOR指令运算组成；  
5. 免费，任何人都可以免费使用不需要缴纳版权费；  
6. BlowFish 每次只能加密和解密8字节数据；

接下来就是最重要的部分，Blowfish加密算法的实现：

```
package cn.bridgeli.encrypt;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
import java.util.Base64;

public class BlowfishUtil {

    private static final String ALGORITHM = "Blowfish";
    // 使用 CBC 模式保证安全性，PKCS5Padding 处理数据填充
    private static final String TRANSFORMATION = "Blowfish/CBC/PKCS5Padding";

    /**
     * 加密方法
     * @param plainText 需要加密的明文
     * @param key 密钥字符串
     * @return 返回 Base64 编码的字符串，格式为 "IV:密文"
     */
    public static String encrypt(String plainText, String key) throws Exception {
        // 1. 根据密钥字符串生成密钥对象
        SecretKeySpec secretKeySpec = new SecretKeySpec(key.getBytes("UTF-8"), ALGORITHM);
        
        // 2. 生成随机的 8 字节 IV (Blowfish 的块大小是 64 位 = 8 字节)
        byte[] iv = new byte[8];
        new SecureRandom().nextBytes(iv);
        IvParameterSpec ivSpec = new IvParameterSpec(iv);

        // 3. 初始化 Cipher 并执行加密
        Cipher cipher = Cipher.getInstance(TRANSFORMATION);
        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, ivSpec);
        byte[] encryptedBytes = cipher.doFinal(plainText.getBytes("UTF-8"));

        // 4. 将 IV 和真实的密文拼接在一起，并用 Base64 编码返回
        // 注意：IV 不需要保密，但解密时必须使用相同的 IV
        byte[] combined = new byte[iv.length + encryptedBytes.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(encryptedBytes, 0, combined, iv.length, encryptedBytes.length);

        return Base64.getEncoder().encodeToString(combined);
    }

    /**
     * 解密方法
     * @param encryptedData 传入加密后的 Base64 字符串 (格式为 "IV:密文")
     * @param key 密钥字符串
     * @return 解密后的明文
     */
    public static String decrypt(String encryptedData, String key) throws Exception {
        // 1. 先进行 Base64 解码
        byte[] combined = Base64.getDecoder().decode(encryptedData);

        // 2. 从数据的最前面提取出 8 字节的 IV
        byte[] iv = new byte[8];
        System.arraycopy(combined, 0, iv, 0, iv.length);
        IvParameterSpec ivSpec = new IvParameterSpec(iv);

        // 3. 提取出剩余部分的真实密文
        byte[] encryptedBytes = new byte[combined.length - iv.length];
        System.arraycopy(combined, iv.length, encryptedBytes, 0, encryptedBytes.length);

        // 4. 初始化 Cipher 并执行解密
        SecretKeySpec secretKeySpec = new SecretKeySpec(key.getBytes("UTF-8"), ALGORITHM);
        Cipher cipher = Cipher.getInstance(TRANSFORMATION);
        cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivSpec);
        byte[] decryptedBytes = cipher.doFinal(encryptedBytes);

        return new String(decryptedBytes, "UTF-8");
    }

    // 测试主方法
    public static void main(String[] args) {
        try {
            String key = "MySecretKey123"; // 密钥长度建议在 8-56 字节之间
            String text = "Hello, Blowfish! 这是一段测试文本。";

            System.out.println("原始明文: " + text);

            // 连续加密两次，你会发现输出的密文是完全不一样的
            String encrypted1 = encrypt(text, key);
            String encrypted2 = encrypt(text, key);
            
            System.out.println("第一次加密结果: " + encrypted1);
            System.out.println("第二次加密结果: " + encrypted2);

            // 验证解密是否正常
            String decrypted = decrypt(encrypted1, key);
            System.out.println("解密后的明文: " + decrypted);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```