---
layout: post
title:  "Java常见安全算法"
categories: java
tags: java 安全算法
---

* content
{:toc}


本文整理了常见的安全算法，包括MD5、SHA、DES、AES、RSA等，并写了完整的工具类（Java 版），工具类包含测试。

<!--more-->

## 一、数字摘要算法

> 数字摘要也称为消息摘要，它是一个唯一对应一个消息或文本的固定长度的值，它由一个单向Hash函数对消息进行计算而产生。如果消息在传递的途中改变了，接收者通过对收到消息采用相同的Hash重新计算，新产生的摘要与原摘要进行比较，就可知道消息是否被篡改了，因此消息摘要能够验证消息的完整性。消息摘要采用单向Hash函数将需要计算的内容"摘要"成固定长度的串，这个串亦称为数字指纹。这个串有固定的长度，且不同的明文摘要成密文，其结果总是不同的(相对的)，而同样的明文其摘要必定一致。这样这串摘要便可成为验证明文是否是"真身"的"指纹"了。


### 1. Md5

MD5即Message Digest Algorithm 5(信息摘要算法5)，是数字摘要算法一种实现，用于确保信息传输完整性和一致性，摘要长度为128位。 MD5由MD4、 MD3、 MD2改进而来，主要增强算法复杂度和不可逆性，该算法因其普遍、稳定、快速的特点，在产业界得到了极为广泛的使用，目前主流的编程语言普遍都已有MD5算法实现。

```
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

/**
 * Message Digest Algorithm 5(信息摘要算法5)
 */
public class MD5Util {
	/**
	 * Constructs the MD5Util object and sets the string whose MD5Util is to be
	 * computed.
	 * 
	 * @param inStr
	 *    the <code>String</code> whose MD5Util is to be computed
	 */
	
	
	public final static String COMMON_KEY="zhongzhuoxin#@!321";
	public MD5Util() {

	}

	public final static String str2MD5(String inStr) {
		char hexDigits[] = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
				'a', 'b', 'c', 'd', 'e', 'f' };
		try {
			byte[] strTemp = inStr.getBytes("UTF-8");
			MessageDigest mdTemp = MessageDigest.getInstance("MD5");
			mdTemp.update(strTemp);
			byte[] md = mdTemp.digest();
			int j = md.length;
			char str[] = new char[j * 2];
			int k = 0;
			for (int i = 0; i < j; i++) {
				byte byte0 = md[i];
				str[k++] = hexDigits[byte0 >>> 4 & 0xf];
				str[k++] = hexDigits[byte0 & 0xf];
			}
			return new String(str);
		} catch (Exception e) {
			return null;
		}
	}
	
	
	

	//--MD5Util
	private static final char HEX_DIGITS[] = { '0', '1', '2', '3', '4', '5',
			'6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };

	public static String toHexString(byte[] b) { // String to byte
		StringBuilder sb = new StringBuilder(b.length * 2);
		for (int i = 0; i < b.length; i++) {
			sb.append(HEX_DIGITS[(b[i] & 0xf0) >>> 4]);
			sb.append(HEX_DIGITS[b[i] & 0x0f]);
		}
		return sb.toString();
	}

	public static String AndroidMd5(String s) {
		try {
			// Create MD5Util Hash
			MessageDigest digest = MessageDigest
					.getInstance("MD5");
			digest.update(s.getBytes());
			byte messageDigest[] = digest.digest();

			return toHexString(messageDigest);
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		}

		return "";
	}

	public static void main(String[] args) {

		String m = MD5Util.str2MD5("swwwwwwwwwwdkinner");

		System.out.print(m.length() + "    ");
		System.out.println(m);

	}
}

```

###2.SHA

SHA的全称是Secure Hash Algorithm，即安全散列算法。 1993年，安全散列算法(SHA)由美国国家标准和技术协会（NIST)提出，并作为联邦信息处理标准(FIPS PUB 180)公布， 1995年又发布了一个修订版FIPS PUB 180-1，通常称之为SHA-1。 SHA-1是基于MD4算法的，现在已成为公认的最安全的散列算法之一，并被广泛使用。SHA-1算法生成的摘要信息的长度为160位，由于生成的摘要信息更长，运算的过程更加复杂，在相同的硬件上， SHA-1的运行速度比MD5更慢，但是也更为安全。

```


import com.google.common.base.Strings;

import java.security.MessageDigest;

/**
 * SHA的全称是Secure Hash Algorithm，即安全散列算法
 * Created by fangzhipeng on 2017/3/21.
 */
public class SHAUtil {

    /**
     * 定义加密方式
     */
    private final static String KEY_SHA = "SHA";
    private final static String KEY_SHA1 = "SHA-1";
    /**
     * 全局数组
     */
    private final static String[] hexDigits = { "0", "1", "2", "3", "4", "5",
            "6", "7", "8", "9", "a", "b", "c", "d", "e", "f" };

    /**
     * 构造函数
     */
    public SHAUtil() {

    }

    /**
     * SHA 加密
     * @param data 需要加密的字节数组
     * @return 加密之后的字节数组
     * @throws Exception
     */
    public static byte[] encryptSHA(byte[] data) throws Exception {
        // 创建具有指定算法名称的信息摘要
//        MessageDigest sha = MessageDigest.getInstance(KEY_SHA);
        MessageDigest sha = MessageDigest.getInstance(KEY_SHA1);
        // 使用指定的字节数组对摘要进行最后更新
        sha.update(data);
        // 完成摘要计算并返回
        return sha.digest();
    }

    /**
     * SHA 加密
     * @param data 需要加密的字符串
     * @return 加密之后的字符串
     * @throws Exception
     */
    public static String encryptSHA(String data) throws Exception {
        // 验证传入的字符串
        if (Strings.isNullOrEmpty(data)) {
            return "";
        }
        // 创建具有指定算法名称的信息摘要
        MessageDigest sha = MessageDigest.getInstance(KEY_SHA);
        // 使用指定的字节数组对摘要进行最后更新
        sha.update(data.getBytes());
        // 完成摘要计算
        byte[] bytes = sha.digest();
        // 将得到的字节数组变成字符串返回
        return byteArrayToHexString(bytes);
    }

    /**
     * 将一个字节转化成十六进制形式的字符串
     * @param b 字节数组
     * @return 字符串
     */
    private static String byteToHexString(byte b) {
        int ret = b;
        //System.out.println("ret = " + ret);
        if (ret < 0) {
            ret += 256;
        }
        int m = ret / 16;
        int n = ret % 16;
        return hexDigits[m] + hexDigits[n];
    }

    /**
     * 转换字节数组为十六进制字符串
     * @param bytes 字节数组
     * @return 十六进制字符串
     */
    private static String byteArrayToHexString(byte[] bytes) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < bytes.length; i++) {
            sb.append(byteToHexString(bytes[i]));
        }
        return sb.toString();
    }

    /**
     * 测试方法
     * @param args
     */
    public static void main(String[] args) throws Exception {
        String key = "123";
        System.out.println(encryptSHA(key));
    }
}

```

## 二、对称加密

> 对称加密算法是应用较早的加密算法，技术成熟。在对称加密算法中，数据发送方将明文(原始数据)和加密密钥一起经过特殊加密算法处理后，生成复杂的加密密文进行发送，数据接收方收到密文后，若想读取原文，则需要使用加密使用的密钥及相同算法的逆算法对加密的密文进行解密，才能使其恢复成可读明文。在对称加密算法中，使用的密钥只有一个，发送和接收双方都使用这个密钥对数据进行加密和解密，这就要求加密和解密方事先都必须知道加密的密钥。

### 1. DES算法

1973 年，美国国家标准局(NBS)在认识到建立数据保护标准既明显又急迫的情况下，开始征集联邦数据加密标准的方案。 1975 年3月17日， NBS公布了IBM公司提供的密码算法，以标准建议的形式在全国范围内征求意见。经过两年多的公开讨论之后， 1977 年7月15日， NBS宣布接受这建议，作为联邦信息处理标准46 号数据加密标准(Data Encryptin Standard)，即DES正式颁布，供商业界和非国防性政府部门使用。DES算法属于对称加密算法，明文按64位进行分组，密钥长64位，但事实上只有56位参与DES
运算(第8、 16、 24、 32、 40、 48、 56、 64位是校验位，使得每个密钥都有奇数个1),分组后的明文和56位的密钥按位替代或交换的方法形成密文。由于计算机运算能力的增强，原版DES密码的密钥长度变得容易被暴力破解，因此演变出了3DES算法。 3DES是DES向AES过渡的加密算法，它使用3条56位的密钥对数据进行三次加密，是DES的一个更安全的变形

```


import java.io.IOException;
import java.security.SecureRandom;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;


import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

/**
 * Data Encryptin Standard
 * 数据加密标准
 */
public class DESUtil {


    private final static String DES = "DES";

    /**
     * Description 根据键值进行加密
     *
     * @param data
     * @param key  加密键byte数组
     * @return
     * @throws Exception
     */
    public static String encrypt(String data, String key) throws Exception {
        byte[] bt = encrypt(data.getBytes(), key.getBytes());
        String strs = new BASE64Encoder().encode(bt);
        return strs;
    }

    /**
     * Description 根据键值进行解密
     *
     * @param data
     * @param key  加密键byte数组
     * @return
     * @throws IOException
     * @throws Exception
     */
    public static String decrypt(String data, String key) throws Exception,
            Exception {
        if (data == null)
            return null;
        BASE64Decoder decoder = new BASE64Decoder();
        byte[] buf = decoder.decodeBuffer(data);
        byte[] bt = decrypt(buf, key.getBytes());
        return new String(bt);
    }

    /**
     * Description 根据键值进行加密
     *
     * @param data
     * @param key  加密键byte数组
     * @return
     * @throws Exception
     */
    private static byte[] encrypt(byte[] data, byte[] key) throws Exception {
        // 生成一个可信任的随机数源
        SecureRandom sr = new SecureRandom();

        // 从原始密钥数据创建DESKeySpec对象
        DESKeySpec dks = new DESKeySpec(key);

        // 创建一个密钥工厂，然后用它把DESKeySpec转换成SecretKey对象
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(DES);
        SecretKey securekey = keyFactory.generateSecret(dks);

        // Cipher对象实际完成加密操作
        Cipher cipher = Cipher.getInstance(DES);

        // 用密钥初始化Cipher对象
        cipher.init(Cipher.ENCRYPT_MODE, securekey, sr);

        return cipher.doFinal(data);
    }


    /**
     * Description 根据键值进行解密
     *
     * @param data
     * @param key  加密键byte数组
     * @return
     * @throws Exception
     */
    private static byte[] decrypt(byte[] data, byte[] key) throws Exception {
        // 生成一个可信任的随机数源
        SecureRandom sr = new SecureRandom();

        // 从原始密钥数据创建DESKeySpec对象
        DESKeySpec dks = new DESKeySpec(key);

        // 创建一个密钥工厂，然后用它把DESKeySpec转换成SecretKey对象
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(DES);
        SecretKey securekey = keyFactory.generateSecret(dks);

        // Cipher对象实际完成解密操作
        Cipher cipher = Cipher.getInstance(DES);

        // 用密钥初始化Cipher对象
        cipher.init(Cipher.DECRYPT_MODE, securekey, sr);

        return cipher.doFinal(data);
    }

    public static void main(String[]args)throws Exception{
       String  sStr=encrypt("122222112222:12343232323:jajwwwwslwskwkkwksk","wew2323w233321ws233w");
       System.out.println(sStr);
       String mStr=decrypt(sStr,"wew2323w233321ws233w");
       System.out.println(mStr);
    }
}

```

### 2. AES

AES的全称是Advanced Encryption Standard，即高级加密标准，该算法由比利时密码学家Joan Daemen和Vincent Rijmen所设计，结合两位作者的名字，又称Rijndael加密算法，是美国联邦政府采用的一种对称加密标准，这个标准用来替代原先的DES算法，已经广为全世界所使用，已然成为对称加密算法中最流行的算法之一。AES算法作为新一代的数据加密标准汇聚了强安全性、高性能、高效率、易用和灵活等优
点，设计有三个密钥长度:128,192,256位，比DES算法的加密强度更高，更为安全。

```

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.util.Base64;
import java.util.Scanner;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

/**
 * Created by fangzhipeng on 2017/3/21.
 */
public class AESUtil {

    static  byte[]  key = "w@#$4@#$s^&3*&^4".getBytes();
    final static String algorithm="AES";

    public static String encrypt(String data){

        byte[] dataToSend = data.getBytes();
        Cipher c = null;
        try {
            c = Cipher.getInstance(algorithm);
        } catch (NoSuchAlgorithmException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        SecretKeySpec k =  new SecretKeySpec(key, algorithm);
        try {
            c.init(Cipher.ENCRYPT_MODE, k);
        } catch (InvalidKeyException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        byte[] encryptedData = "".getBytes();
        try {
            encryptedData = c.doFinal(dataToSend);
        } catch (IllegalBlockSizeException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (BadPaddingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        byte[] encryptedByteValue =     Base64.getEncoder().encode(encryptedData);
        return  new String(encryptedByteValue);//.toString();
    }

    public static String decrypt(String data){

        byte[] encryptedData  =  Base64.getDecoder().decode(data);
        Cipher c = null;
        try {
            c = Cipher.getInstance(algorithm);
        } catch (NoSuchAlgorithmException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        SecretKeySpec k =
                new SecretKeySpec(key, algorithm);
        try {
            c.init(Cipher.DECRYPT_MODE, k);
        } catch (InvalidKeyException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        }
        byte[] decrypted = null;
        try {
            decrypted = c.doFinal(encryptedData);
        } catch (IllegalBlockSizeException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (BadPaddingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return new String(decrypted);
    }

    public static void main(String[] args){
        String password=encrypt("12233440988:1239874389888:dd333");
        System.out.println(password);
        System.out.println(decrypt(password));
    }
}


```

##三、非对称加密

> 非对称加密算法又称为公开密钥加密算法，它需要两个密钥，一个称为公开密钥(public key)，即公钥，另一个称为私有密钥(private key)，即私钥。公钥与私钥需要配对使用，如果用公钥对数据进行加密，只有用对应的私钥才能进行解密，而如果使用私钥对数据进行加密，那么只有用对应的公钥才能进行解密。因为加密和解密使用的是两个不同的密钥，所以这种算法称为非对称加密算法。非对称加密算法实现机密信息交换的基本过程是：甲方生成一对密钥并将其中的一把作为公钥向其它人公开，得到该公钥的乙方使用该密钥对机密信息进行加密后再发送给甲方，甲方再使用自己保存的另一把专用密钥，即私钥，对加密后的信息进行解密。

### RSA

RSA非对称加密算法是1977年由Ron Rivest、 Adi Shamirh和LenAdleman开发的， RSA取名来自开发他们三者的名字。 RSA是目前最有影响力的非对称加密算法，它能够抵抗到目前为止已知的所有密码攻击，已被ISO推荐为公钥数据加密标准。 RSA算法基于一个十分简单的数论事实：将两个大素数相乘十分容易，但反过来想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。

```

/**
 * Created by fangzhipeng on 2017/3/21.
 * RSA ：RSA非对称加密算法是1977年由Ron Rivest、 Adi Shamirh和LenAdleman开发   *  的， RSA取名来
 *  自开发他们三者的名字。
 * 参考：http://blog.csdn.net/wangqiuyun/article/details/42143957
 */

import java.io.*;
import java.security.InvalidKeyException;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;

import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
public class RSAUtil {


    /**
     * 字节数据转字符串专用集合
     */
    private static final char[] HEX_CHAR = { '0', '1', '2', '3', '4', '5', '6',
            '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f' };

    /**
     * 随机生成密钥对
     */
    public static void genKeyPair(String filePath) {
        // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
        KeyPairGenerator keyPairGen = null;
        try {
            keyPairGen = KeyPairGenerator.getInstance("RSA");
        } catch (NoSuchAlgorithmException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        // 初始化密钥对生成器，密钥大小为96-1024位
        keyPairGen.initialize(1024,new SecureRandom());
        // 生成一个密钥对，保存在keyPair中
        KeyPair keyPair = keyPairGen.generateKeyPair();
        // 得到私钥
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        // 得到公钥
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        try {
            // 得到公钥字符串
            // 得到私钥字符串
            String privateKeyString =new String( Base64.getEncoder().encode(privateKey.getEncoded()));
            String publicKeyString =new String( Base64.getEncoder().encode(publicKey.getEncoded()));
            // 将密钥对写入到文件

            File file1=new File(filePath + "publicKey.keystore");
            File file2=new File(filePath + "privateKey.keystore");
            if(!file1.exists()) {
                file1.createNewFile();
            }
            if(!file2.exists()) {
                file2.createNewFile();
            }
            FileWriter pubfw = new FileWriter(filePath + "/publicKey.keystore");
            FileWriter prifw = new FileWriter(filePath + "/privateKey.keystore");
            BufferedWriter pubbw = new BufferedWriter(pubfw);
            BufferedWriter pribw = new BufferedWriter(prifw);
            pubbw.write(publicKeyString);
            pribw.write(privateKeyString);
            pubbw.flush();
            pubbw.close();
            pubfw.close();
            pribw.flush();
            pribw.close();
            prifw.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 从文件中输入流中加载公钥
     *
     * @param
     *
     * @throws Exception
     *             加载公钥时产生的异常
     */
    public static String loadPublicKeyByFile(String path) throws Exception {
        try {
            BufferedReader br = new BufferedReader(new FileReader(path
                    + "/publicKey.keystore"));
            String readLine = null;
            StringBuilder sb = new StringBuilder();
            while ((readLine = br.readLine()) != null) {
                sb.append(readLine);
            }
            br.close();
            return sb.toString();
        } catch (IOException e) {
            throw new Exception("公钥数据流读取错误");
        } catch (NullPointerException e) {
            throw new Exception("公钥输入流为空");
        }
    }

    /**
     * 从字符串中加载公钥
     *
     * @param publicKeyStr
     *            公钥数据字符串
     * @throws Exception
     *             加载公钥时产生的异常
     */
    public static RSAPublicKey loadPublicKeyByStr(String publicKeyStr)
            throws Exception {
        try {
            byte[] buffer = Base64.getDecoder().decode(publicKeyStr);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            X509EncodedKeySpec keySpec = new X509EncodedKeySpec(buffer);
            return (RSAPublicKey) keyFactory.generatePublic(keySpec);
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此算法");
        } catch (InvalidKeySpecException e) {
            throw new Exception("公钥非法");
        } catch (NullPointerException e) {
            throw new Exception("公钥数据为空");
        }
    }

    /**
     * 从文件中加载私钥
     *
     * @param
     *
     * @return 是否成功
     * @throws Exception
     */
    public static String loadPrivateKeyByFile(String path) throws Exception {
        try {
            BufferedReader br = new BufferedReader(new FileReader(path
                    + "/privateKey.keystore"));
            String readLine = null;
            StringBuilder sb = new StringBuilder();
            while ((readLine = br.readLine()) != null) {
                sb.append(readLine);
            }
            br.close();
            return sb.toString();
        } catch (IOException e) {
            throw new Exception("私钥数据读取错误");
        } catch (NullPointerException e) {
            throw new Exception("私钥输入流为空");
        }
    }

    public static RSAPrivateKey loadPrivateKeyByStr(String privateKeyStr)
            throws Exception {
        try {
            byte[] buffer = Base64.getDecoder().decode(privateKeyStr);
            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(buffer);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            return (RSAPrivateKey) keyFactory.generatePrivate(keySpec);
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此算法");
        } catch (InvalidKeySpecException e) {
            throw new Exception("私钥非法");
        } catch (NullPointerException e) {
            throw new Exception("私钥数据为空");
        }
    }

    /**
     * 公钥加密过程
     *
     * @param publicKey
     *            公钥
     * @param plainTextData
     *            明文数据
     * @return
     * @throws Exception
     *             加密过程中的异常信息
     */
    public static byte[] encrypt(RSAPublicKey publicKey, byte[] plainTextData)
            throws Exception {
        if (publicKey == null) {
            throw new Exception("加密公钥为空, 请设置");
        }
        Cipher cipher = null;
        try {
            // 使用默认RSA
            cipher = Cipher.getInstance("RSA");
            // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            byte[] output = cipher.doFinal(plainTextData);
            return output;
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此加密算法");
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
            return null;
        } catch (InvalidKeyException e) {
            throw new Exception("加密公钥非法,请检查");
        } catch (IllegalBlockSizeException e) {
            throw new Exception("明文长度非法");
        } catch (BadPaddingException e) {
            throw new Exception("明文数据已损坏");
        }
    }

    /**
     * 私钥加密过程
     *
     * @param privateKey
     *            私钥
     * @param plainTextData
     *            明文数据
     * @return
     * @throws Exception
     *             加密过程中的异常信息
     */
    public static byte[] encrypt(RSAPrivateKey privateKey, byte[] plainTextData)
            throws Exception {
        if (privateKey == null) {
            throw new Exception("加密私钥为空, 请设置");
        }
        Cipher cipher = null;
        try {
            // 使用默认RSA
            cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE, privateKey);
            byte[] output = cipher.doFinal(plainTextData);
            return output;
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此加密算法");
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
            return null;
        } catch (InvalidKeyException e) {
            throw new Exception("加密私钥非法,请检查");
        } catch (IllegalBlockSizeException e) {
            throw new Exception("明文长度非法");
        } catch (BadPaddingException e) {
            throw new Exception("明文数据已损坏");
        }
    }

    /**
     * 私钥解密过程
     *
     * @param privateKey
     *            私钥
     * @param cipherData
     *            密文数据
     * @return 明文
     * @throws Exception
     *             解密过程中的异常信息
     */
    public static byte[] decrypt(RSAPrivateKey privateKey, byte[] cipherData)
            throws Exception {
        if (privateKey == null) {
            throw new Exception("解密私钥为空, 请设置");
        }
        Cipher cipher = null;
        try {
            // 使用默认RSA
            cipher = Cipher.getInstance("RSA");
            // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
            cipher.init(Cipher.DECRYPT_MODE, privateKey);
            byte[] output = cipher.doFinal(cipherData);
            return output;
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此解密算法");
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
            return null;
        } catch (InvalidKeyException e) {
            throw new Exception("解密私钥非法,请检查");
        } catch (IllegalBlockSizeException e) {
            throw new Exception("密文长度非法");
        } catch (BadPaddingException e) {
            throw new Exception("密文数据已损坏");
        }
    }

    /**
     * 公钥解密过程
     *
     * @param publicKey
     *            公钥
     * @param cipherData
     *            密文数据
     * @return 明文
     * @throws Exception
     *             解密过程中的异常信息
     */
    public static byte[] decrypt(RSAPublicKey publicKey, byte[] cipherData)
            throws Exception {
        if (publicKey == null) {
            throw new Exception("解密公钥为空, 请设置");
        }
        Cipher cipher = null;
        try {
            // 使用默认RSA
            cipher = Cipher.getInstance("RSA");
            // cipher= Cipher.getInstance("RSA", new BouncyCastleProvider());
            cipher.init(Cipher.DECRYPT_MODE, publicKey);
            byte[] output = cipher.doFinal(cipherData);
            return output;
        } catch (NoSuchAlgorithmException e) {
            throw new Exception("无此解密算法");
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
            return null;
        } catch (InvalidKeyException e) {
            throw new Exception("解密公钥非法,请检查");
        } catch (IllegalBlockSizeException e) {
            throw new Exception("密文长度非法");
        } catch (BadPaddingException e) {
            throw new Exception("密文数据已损坏");
        }
    }

    /**
     * 字节数据转十六进制字符串
     *
     * @param data
     *            输入数据
     * @return 十六进制内容
     */
    public static String byteArrayToString(byte[] data) {
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < data.length; i++) {
            // 取出字节的高四位 作为索引得到相应的十六进制标识符 注意无符号右移
            stringBuilder.append(HEX_CHAR[(data[i] & 0xf0) >>> 4]);
            // 取出字节的低四位 作为索引得到相应的十六进制标识符
            stringBuilder.append(HEX_CHAR[(data[i] & 0x0f)]);
            if (i < data.length - 1) {
                stringBuilder.append(' ');
            }
        }
        return stringBuilder.toString();
    }



    public static void main(String[] args) throws Exception {
        String filepath="F:/temp/";
        File file=new File(filepath);
        if(!file.exists()){
            file.mkdir();
        }
        genKeyPair(filepath);
        System.out.println("--------------公钥加密私钥解密过程-------------------");
        String plainText="1223333323:8783737321232:dewejj28i33e92hhsxxxx";
        //公钥加密过程
        byte[] cipherData=encrypt(loadPublicKeyByStr(loadPublicKeyByFile(filepath)),plainText.getBytes());
        String cipher=new String(Base64.getEncoder().encode(cipherData));
        //私钥解密过程
        byte[] res=decrypt(loadPrivateKeyByStr(loadPrivateKeyByFile(filepath)), Base64.getDecoder().decode(cipher));
        String restr=new String(res);
        System.out.println("原文："+plainText);
        System.out.println("加密密文："+cipher);
        System.out.println("解密："+restr);
        System.out.println();
    }
}

```

注： 文字部分复制了《大型电商分布式系统实践 第一版 讲师 陈康贤》的第三课。代码来源于自己的整理，全部测试通过，应该没有坑。