---
layout: post
title:  "intro to cryptography"
date:   2022-03-30
categories: crypto
---

Last meeting we learned about [classical cryptography](https://docs.google.com/presentation/d/1YQ6PR15Rls3ZgGVhtXbcRV90DWAPi6sPXcbAvb7-C-w/edit)!

There were four challenges, which can be accessed under the Classical Cryptography set [here](https://blairsec.clamchowder.repl.co/):
- [Challenge 1](#challenge-1)
- [Challenge 2](#challenge-2)
- [Challenge 3](#challenge-3)
- [Challenge 4](#challenge-4)

### Challenge 1
We have a Caesar Cipher with the ciphertext `qvamzbaitilxcvpmzm`. The `ROT13` command in CyberChef can do a Caesar Cipher encryption/decryption for us, and we can check every key until we get some recognizable text. A shift of 18 does the trick, giving the plaintext `insertsaladpunhere`.

[CyberChef link](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,18)&input=cXZhbXpiYWl0aWx4Y3ZwbXpt)

### Challenge 2
A Vigenere ciphertext is given:
```
llgpzoxlwvgwzxacjmuudmmfghqzvvvpqtvceotjhlcvvbbaliznsgnqariujmkgwwqzzvmcjaqpvvvywwcltqifwvuvraxbgrvbvtxrlitmfntiwcyiilbrwqrffglyxstgfnimdccfgptzwxkwjcuqlmvokqhl
```
as well as a crib of `vigenere`. Because of properties of subtraction, if we decrypt the right part of the ciphertext with the crib, we'll see a portion of the key. At an offset of 3 letters from the start, the decryption of the ciphertext with the crib is `uritysec`, which suggests that the key is `security`. This indeed decrypts the ciphertext into English. 

[CyberChef link](https://gchq.github.io/CyberChef/#recipe=Vigen%C3%A8re_Decode('security')&input=bGxncHpveGx3dmd3enhhY2ptdXVkbW1mZ2hxenZ2dnBxdHZjZW90amhsY3Z2YmJhbGl6bnNnbnFhcml1am1rZ3d3cXp6dm1jamFxcHZ2dnl3d2NsdHFpZnd2dXZyYXhiZ3J2YnZ0eHJsaXRtZm50aXdjeWlpbGJyd3FyZmZnbHl4c3RnZm5pbWRjY2ZncHR6d3hrd2pjdXFsbXZva3FobA)

### Challenge 3

A long Vigenere ciphertext is given:
```
oclnrnwotlbqlpyexbvptqgygtxbanihatpenhauhoypuljfwhmcfllnqodmbbzvxrfcmsebllapnqlhmclviyxilvvbvluscofkbbqddybchsfoklgowhvtnsszniwuelxvwyhijaatxwlowhnuaymccfruuzevthdpneumyluoayhdksuhvgwpolksopuvlohgrfdhhoauldxbllydwjwyxilvvbvluscofktvwzdccissmmjldlumauusjlvtnraunssugicgfvuokpwujhavowuhqpjgkbqtgyvovselbbaujlngkkjguvudmsylosjhwebpwjbiklllupwtfslpqgjuaymbgawhjhqvvfwholhojlhcaujtxifkffkadnmhzptwktblrtwupkelhtrbydpawudnmbgaxokalnpaqwssuprubhatfcfslnnwzhwslvpoesgugfgtvulvkpnddlwhrbyzbbviuawqzvvhqvxwxidkohmugeaglhorwchrhcfl
```
The first step is to find the most likely key length. We'll use the python3 interpreter on [tio.run](https://tio.run/#python3) for this. The ciphertext can go into the input field and we can retrieve it with a call to `input()`. We'll also want a way to calculate index of coincidence, which we can code up according to the formula listed on Wikipdia. 
```python
from collections import Counter
ct = input()
def ic(message):
	N = len(message)
	return 26 * sum(freq*(freq-1) for letter, freq in Counter(message).items())/(N * (N-1))
```
Now for each key length `n`, we'll want to find the average index of coincidence across all messages constructed by taking every `n`th character from the original message, which ensures that each of the smaller messages is encrypted by the same key byte. Code below:
```python
for keylen in range(1,31):
	sum_ic = 0
	for offset in range(keylen):
		sum_ic += ic(ct[offset::keylen])
	avg_ic = sum_ic/keylen
	print(keylen, avg_ic, sep='\t')
```
Here's the output of this code:
![ic.png](/writeups/assets/images/03-30-22/ic.png)
The index of coincidence of English is about 1.73, and we can see that a key length of 10 comes pretty close, at 1.69. So, we can be pretty sure that the length of the key is 10.

Now, for actually finding the key, we'll pick key letters that make the frequency distribution of the letters they encode as close as possible to the frequency distribution of English letters, which we'll rip off of Wikipdia. We'll take "close" to mean the least sum of squared differences between frequencies. Code below:
```python
from string import ascii_lowercase as alphabet
freq_english = {"a":8.2,"b":1.5,"c":2.7,"d":4.7,"e":13,"f":2.2,"g":2,"h":6.2,"i":6.9,"j":0.16,"k":0.81,"l":4.0,"m":2.7,"n":6.7,"o":7.8,"p":1.9,"q":0.11,"r":5.9,"s":6.2,"t":9.6,"u":2.7,"v":0.97,"w":2.4,"x":0.15,"y":2,"z":0.078}
keylen = 10
for offset in range(keylen):
	c = ct[offset::keylen]
	best = -1
	leastsquares = float('inf')
	for shift in range(26):
		f = {alphabet[(alphabet.find(i)-shift+26)%26]: c.count(i)/len(c) for i in alphabet}
		sumsquares = sum((f[i]-freq_english[i])**2 for i in alphabet)
		if sumsquares < leastsquares:
			leastsquares = sumsquares
			best = shift
	print(alphabet[best],end='')
```
This gives the output `boshdajosh`, which seems like it should be the key since it produces a decryption of
```
notgonnabeactiveondiscordtonightimmeetingagirlarealoneinhalfanhourwouldntexpectalotofyoutounderstandanywaysopleasedontdmmeaskingmewhereiamimwiththegirlokyoullmostlikelygetairedbecauseillbewiththegirlagainidontexpectyoutounderstandshesactuallyreallyinterestedinmeanditsnotasituationicanpassupforsomemeaninglessdiscorddegeneratesbecauseillbemeetingagirlnotthatyoureallyaregoingtounderstandthisismylifenowmeetingwomenandnotwastingmyprecioustimeonlineihavetomoveonfromsuchsimplethingsandbranchoutyouwouldntunderstandeveryone
```
and indeed that is our answer.

[Full code on tio.run](https://tio.run/##fVPJbuM4ED1bXyFoMIidVpzY6c4iTE5zzw9kgoCiuEkUSYmb5EF/e6Yo20nQDcxFJKtevXq1yMyOa3X7/k5H3edYS0mwE1rZXPRGjy7/W3vlyJhhlz/lQhnv1pusITQXeN0TaxEjmypbPYNXEvVhylYjcX5U@f4uv8yt79d0JMPl8r3abXKqR8A7YC7zZAPqc6oPjq1wpLfrzeZ6/Qwk62cIPOY2o1DuTWBwQu7E1ZEZ0ieWESlG1rvydpd8K0gNQFB3A4@E1JRa4j6Rx8gFewZ/e0rVYfdyxFbVEfMKVa1QYEe@I/b66ALHounEVuZHWJlbYp4u/nEXm@yPL6KzbGm3dWBi504ji4V4kzqSESNL4J0jaTiqictSi96IYlJYDrn/LVBRPWz3ZVEX1W77oyxwUe2392XRFNX3dBKw35YFTWaAMTjLghfVXXqJdD6WRVtUN9vdXVl06fKwKwuZwm/Koj/RqYSEUxfV/fahLExKB5HDEgkBY1H9SAZ7onZF9bgFRn8iCAn4CJeYDN/LYloiQfG8SDqk5839w8/sNMGnfHeT/f@cUvd/H062qolNS3q1y1aSIOvs4NFILJio1MitL4SiMIhlCywX9Av5/m5ZAJp6e276y/p821KhmrXYXC1R3wD85/7utcrxFqeVBc912n18XGuRaM@hP48r@Ckl/Qpr@iJer77OFN6by8v97/Fp5QTNv1D8lX8tblnbX6r9BCfnqSuL9Oy0pR8lJudrSVTzdHGxeX/XWKpRRe1kPUgzk6kOxg1sZm6qkRIcOUMUR57r2XjZ0sh7TKVUg276uj6EaaS4t6SWEhk1SPDKIOZJyBDqIL3FmnZ1PTTNXGNuqe4k05EHp6w9KBE9kVOIMxctQm6K8Ctw5dHcY0xH7w8kON4YRXw/S6/RzJvOeh5YNFp2VhsfpOZspA3nGnnZTKBjbmIbf1HgQjw0GAtr@76VjfQ98t62EnSMyIMWzwRmNHjdmehbjoKOng@mZV09ODYHHSyRdY18KxXrupb54JvezlLblkdSm9jWopNSehMdtdIMrE111AxF3vIhBOiclly3kmNgcZOgHaUdalTPD8bFztVydNGbjkjuxnpuDIoevMAw6Q5JZdAQQagZfQ1ToRiyKBUPPFoZjCaWeUaZC16GzqimkZEDy6Gug/AoDocQQMUUJ9F0mveeEcRAzhgxHzlM9D8)

By the way, there are websites (namely [this one](https://www.guballa.de/vigenere-solver)) that can do this all for you.

### Challenge 4

This challenge goes similarly to the previous one. The exact same code for index of coincidence still works, and yields a key length of 29. However, in this case we can use a bit of a shortcut for finding the key bytes. By far the most common character in English text is the space character, so we can just pick key bytes that maximize the number of spaces in the output. Code below:
```python
keylen = 29
key = []
for i in range(keylen):
	c = ct[i::keylen]
	key.append(max(range(256),key=lambda x: c.count(ord(' ')^x)))
```
Once you get the key, you can either decode the message in CyberChef, or write a quick XOR implementation in python.
```python
decrypted = []
for i in range(len(ct)):
	decrypted.append(ct[i] ^ key[i % len(key)])
print(bytes(decrypted))
```
This gives us the flag: `UMDCTF{d1d_y0u_use_k4s!sk1_0r_IoC???}`

[Full code on tio.run](https://tio.run/##bZpdjx43coWvrV8xQBBY2jhefrNpwFe59x9wdgOSTXqFWB@RxoH9653nVPeMxlkDsiG983aTrDp1zqnifPzt8R8f3sfff9@fPrx7mB9@/nnNx7cf3n9@ePvu44dPjw//8eGX94/r06v5@PD9w/jtcX3@Vl/9x/r19dv3H395fP3mzatz7Ye38/W79flz/2m9@e7VVz/w5Z/X@@ePXn31aT3@8un9QygPf3n4/Mu71/vT@p@/2P//3b952B8@8f1HFvrmQZ89vH3/tPLzO759@7jefWa9v77@gZe8/oEHr7U/fnr7/vG/3k5@yNp613@v31heb/nU3/@0XvtvotfPvmJpvsjuHP/QNz/s/Xk9fvnm9aR99@nL//a9Tjcff7y@@91313f@xqm@6v/70/W@67t/vX7ED2xP99u@ebi@9s3D5/Xx@6//8/HrN6/@5cWmX7269/v9Q2j6O3/58W@vtL23f7Yzrcd23j7v5NVX/OXb/vHjen@@ftd/fX09EXJ58w0/@f7n/m6c/eHX7x7mt1Nhff3h0/n664ev3/z91zcEkSjOT799fFznn66sTM7HN1r6@YtPq2kff3v4uyL@49uHf7Ws8/c3ROcKgWHm9fNzb978/rvPOa6YW8gh5COfOzi/jlbSGaPzIfXDx@RW9bNEPqu@h3OVVObptj9WzaXMMn3Mc9dcVzqH27H4M8S@/OnGcjlWl1aofse5SnYz5Z7diimWUEY@cxyH1uTJFg8/Qlln3m4vn45QeXL4EuZYfp/eZ9YczpVeYj59WimntXnShZVWbH2l7dpe6ajJzcJZYhnR77n9yHNVnjw4p@ecnjVnC36F6HbwnJMTrcPPmlONwafg@s5@5tzTXCdPpjo4Z92Nf8@2XAyRNc@eUnMHu201usBhR3T9yMcR/PDHJrCl1J66P0ZJcaV2uBkCEdqcM6W6IrEtvMn5Mx49JLboVup/iFDJx1ossmMgInGcyso@cy4t@VDzQYTSc4R68kSoEedu5zwaKeCcg/dPdpt74JwxdT6btfWR@4w@pcHqrdRaMhneR0o7n4OdZuK4esiLCAUfyMoZph@WT3ZLkIhlSqFVopz8SlrzGOCFrITWZxaGOElljbCcZ01ie3q3clsrzdBKJLZl1xRWb5s4konSR0tCQspnaS4YEs7eOOfwOZ3bOVdb8X77xG6X7RYMCW1kgBOBz6gn4/D1itCsvrLm0JqV3OZjd3Yb29JuhazuUmK3RWsmtpULuyUEc7r1B8SX3XJerZGVSl3EPXaODmDkYREiELXxmTurO4SEFK3KSNHOZC8dhztqdZU1V@btoHeVI9dItkkOez@zHyBBuHVlpprHFySw5gp5cAIfhPu6slflAcqS5jw5G8UKhg7tlhNFKl2RS6qyeIA@drVOcAsKS/A1bk69yGejktl3nRcSwO2yKptCfBg9KkK7g77Ok5PduhGsymLeq2eQUFwuZKUSW8JIBlQrADM70Ed9loN8erJVeU2fiyrz7CjXQYQc6IuJrRybrLAUuG3AJRL9UK7dslTvrNmJ0FKE6ll6DskRobxWSykqtmFalTlyRT5zgYeE@M2TXhgCfcItuwUJWioICcKo8gknEOoM7C70xd1bOsUmcMBIu1R2OzkQVdk9UExkhcrWCX1YAwyv7IInQsrxOgtZaVetlM1uz00oeTK4wZOVfB5ps4uRVflVx4QMLJ/g9iRbq8AeiYrYPZ08SRxj59C9F9AnTqlFFZVZ81BWvtRKOctJfZaLb5tiu4nQSQbIhGEopAxAz9C/PAn2oSTFjnra@nMQgzhcjWu07JXPvAv7DsQrVkNC9OAYRbjOKSSopkJzqgvWBNT5uHWlEMdAbIud07uD@gj2pBiMjKSDlOSbTSi0DFZ5vpTCt6LOSbUP4dbH3jjnJgKFtxwg4TyIcixSCGLjOHlG7WracfBkGSLM4Foe21EJrRbIi0CkvcQm66qVe7cwMSp4IZ4qyHMS8TTu@hT6M7WCirSrshsYSmCoUaiONXMYlOF8oZ/RtAzcwtRZSDjgPrEJVVauCJkKkmT2EVbxRBVgEyeqdlcq1eUdKNdjojZOu239AK9jdDJCFEKD8QkJGEDTcwX3KHQESf4UEjbp9NptCWMMrTk6WRE6qutQCxwwwTi1gnZU6B8eOjlY6uep8wx8QutF50SApCtDuL1VcIqHbo4nZ1afxmC8f5IVGIzKrqzSjKmLdxahwXlYR4pUO2tS9tLPfVhWgvIJm0QHElIWa4Ih2IT3dFN71jROgDUKCpISUtOd1B4Fau5ekw2lXggunIDXOCtMCd8aJ0j5KSgQ406hvQCUQQWeVLbPA3DHsMV9eg8cjy@gVmod5SRC3mILmwgJV2V7OZb7nBWU19ZMHcxhdClvAcFUeSBfgTXhQs6ZhCGe9BbbGGBCdCVnIISQkBVQ6bUmBkVMXaXZUd4HgiSfVVnh7ZgCWCazg8o5A5ELEKR2G4lWytliC2ZZd472lM9S4L4DdQgdJ3UWFQzZ33VGYqvFQIaQkC1CMJDodov7JlkH5@yWuFAr6Cc4PMRgqOCBk5LaH3pSnEB9NQIuXm36jNiqLLowpJKQxpnyKkIb4l@qCdPslAeLopg1qFQg1K3dTvDpk2EIF2OVDcJlxaiBA/2E41OD@xweRoisxpo4VNhEPqH8QVeecGvOWBlLev14idssq/h0zqMWixBZ4UhZihQ6O@Ps5HO89GB/lk/fkQe2Bcm2uCC@KLVGkfBr8pqwyQC3BxgyXeGcHVIUU@M1yzAfP1kTp8qaHh7y2cgknPIm3eVlms1eUrYqc1gkhye9OD4on2Ob2hMN@Nbhu@GYQY0faDoMttHsrnOq4vjv5LUpNlN7XK5vcsYs2i8kyNWY2hvitWaGaOFkzCXUfehJdtsy1QPrKqwhyNVUlLdDe828ZqU@Q8ZDsq8gRTK@RSrMJ7g9UXt2Y5q9YBMpEo5N9UlW/O3e4uVqYnNw8JGMEypnHE4A9ReGnlQQn43Ao/agAsQnmPpaE@VEkdi/MTX6hhs/@axLV4gnj@MYt7nUW1cccTX3hkt1yfzQTheDobxJbrwYhmA21sQ5428vTgiytcQ2mzoQrYLHs1qhR@KcpFgMhtFsqH3my6BffRk1ASnpSe/NvSVzNZvPWTP302WYiB02CP@L76uqbHgoyveZH9rCFBw/zSfwNpBg8JEfKmJQubehTod3Q6XUJ@DHvWFK5GLx1AkkgDD1SKwBqsuprCBptFNQJs4UyZSrobviyUFW6BaK5H92yTp8e6vDSbK6uM/Qt3CpyTT70DlNs1Qr6q46HN9P9QtXfbJZczUglE1Y/6lw6a08mfFgOAjWdPIJz9wX5FKrl8P4kzWlDgCyLOuRmlyqOjoquywBYJqPv3okpJm4Qi1xgITDyaE0E3uxyXE5KfyQzpkRESgcxKO8l6sJQl/Eh4DQqy8r4ttuVXY8If7Wz0qHrg4gXYiXIvHkYc44mRtffAbi02S3YEg9Dx2AM07Ac@E1k7oOPMEei3wGMTW9Jz2ZnBSZGs9VRp@tzpUDt1Tlb4UEdIWW7soKbIJDt27ZukhjMBJjHV1U5zqsL8N@N9ac8tR5@kzP26zKvOI9BChWubrIqrJ1sAmYg9bL3dub10QMxSYJGFaL0DbfhyPkFITbIjT2Nk/dXbR@pdO5JvVl@FuYlHrBAYDlqY7OsnJcvi9dUwHFVocfOCnjIRAv1rQ1Oefh974wJDEOqhU5RpQ3EiFqk92Wuy9jf1XTD40jGgmu@KGWnVXZNj8ktZcFvHpB0UZTL0hln3eVST/xUAjLka/uSkjAt/L2YC51RuqPfGY9ad9Qd7UuNsHfsm9zNakU63QK5eF4MpuP9@qz4QQPl3dFiECiVKa8iTWxpXLGtbiJSUj5mifQKTtPZaN4lhX1ZYPYdhwj5xQS7onLtukHNIAiQTys92Lioifl4yHEckdIPe9gL/uqbOOEbJWNrrgE38qNoyvkcqAh0ZyOqT3lrg5ZXYfUvpqPX7c6yKUmIWE1XE1SZQu3AzaZEAFromrW28@nbtm6SI6AvNFzeWofqEZ1kuC5sa9FPtNLn0BV84fQLzaCchKhKN@BlsEJ9IKwjBhGyutgb1wqbRAYAVB0PX4uy0pkTXWB4fKayFlQ6aKfwUdDvHArRZqa1aArwdShy2FAAqcmaHoV3oTe0J1PHYCXumdNKKpqdWx1kZCwa8YJEbzIYdzdFX32y6w0XEVGs9FP6wX1GgQzyuOK9Sax7TZjhIPgoWE@YThVMx6sDM3e5MatWzbH6GeoYz1P7axWTMsiHTpKpmnk7d7Uf0pnnqZ2oqGl2MJDzrplOYxTM58Tb6VJoVlSdzkpm/J08ZAiRAaS9YKaD4lvpzkpJ/emXtDOWdDsSx0cvX184U2QaLlUSH6Zj392xqAv3M54rHj3n5ciiZUB0KILIbYRXdGa0fqVkaVXNHqga2lNmzFCheC2xWaOcd8O48QZny@yslgHtZd@wrfXOZfqhDXXOMwZ25OACCRQ2ZrCanzVzHkI/O6J449p@QwFBkvdaVIGUz25VDunOh2d7bRzWpXBQ5CqHKNrmvfhWIL0uDj1ZVQZBsHQ5598nzgc9A2ZZfQzGYOpG2/Ch3xfErYPZx7sUnveSK2UevX2lwqqRwJDE/@8zCVhjc0xkj/ISR3AgO@EPuHS8cgTm8iNn/HuP0WeVV0ka3p6Xmm2YivE2zmH8pmjumXjvmGdDhq7p@H2jJq4jOfZ@MVg@B3D7ZbyHoZbdjuENGy7FDaIlO/KVlaCegd2W63/nGjZXuWe38JW10TUOjrNGN09w1BsT6HPMFStypqmJC3dDHZpWbVp1sD3Wf/J2bsEzrqrXHF@yVuE/skPPU8Fzsu94Z4PgEOFqJcmtuod6D15oZqFoaxYpyNdkY8F8ZH0RUlrnwXnAdSI@fmEWzHYAVafu0iKjirTNGtHf7GmaVmzcxIUY@r4HCGrsmdPjeKgZSNs665abiABwGzYVJ46@S@emioTv2IUxGvXtNk4/pqlnuZShVv1SP7uXD29oLpz4XaODWrwdJoK4KSu2TgOY9LEmAdL9IKwothcXQc8tIJwu4375MZVJ3KM9coK1WT3OFmmapiCRIvt7WqsVvBF1MF53R7YOavWhMGurkPelXBefKvKtga4g75rTm39it1YXBPRanPNZd3VsCpb8vWNLsemsOmpc9Ws7@46zqvrUFZYc9@sSS/oVLLNOOHUXcc9Y7ReUAwW5VznKdzUNeHkdnG8TXkG6MP3ST@vJ4kQSND87eYhZ/O@U4oMAcmDjSuf1vM6xfa6Jbnr8w@6wm6TTSI0p9ZEwKZ2LVovaB7smvyaMw7abqxIwd07wAM06JrsWtexdrzuALyyApvgh1g/yZoHzentBgqfkPdLJOh@y3rBTopiOHBNT95EXtODvs6TizXV6WRxnyHe8smaX/I5zaWCRI2ug2oFamW3gfo8xVq6PbA1VSv5dqmwGM7Y4wyXnD4YglnsHmlpUlh1/pPmRnPNZl3HaifRZx0sYbr7T5oqMZicMbzhNDcBt0h7aOo@zW1kqcPtaoyHpCuKLfp52kxqG1M3TZvh/fO6X4ET8JKaChAh9Z/V/C3BpJ/plze5OnRzNUV6ZzOMyG6z5ZMI2T1SoFagEnVXzrlhvX28Zxjmqc33iQykvMiiK0RIsdXNTHqaoN23e7qxsCnPddNm6mCIX1G9IIgnvpcf4jzqIoWjQ61vHup01EFcPRKwMyQsuhFwa/dlmmG4@HRjQddx2GSpqgOQj0/BeiSwtbNN8lFRu1EM1yRCXrNYt@zULZ/0guqdju01PBRT16k4q1ZUi@6Q2hubdLuLhMLAkKfPHren1mx86N4hbzwY5HH3K82mH17Tj7yME7r35jWtymy3hfpEg3O3W2lMqVWZplkxXRM0qjpLIE@b/OruSvcO1yyVUgB9/Jw@O5qWbfkhu8WMz7sFp8uyAt/Oe8rjNPl9nn5M1Se5ExjHi1lNsYnLOISELTNw51Poy7pfwffRbDwhgXMmzaRKuDvXakwtlB@mZXRm7JaaP2EP/zx7Mx9/7/bWVMOQ3Sje8yFDH1uTv1WVhUPu7WnKo841IuD03rc3QamqTbh1S3Jab68eOwq3g2e05o1bde7ObojnrfYUj5Cgm2vOmbWm7lcNCfm63TO@naYruue9uc@QMKTf7HZKy8wZo74gweGdgnUA8rfbJr92@07@hCEMxj2TorJbU@dK7vJ9F2nzvut3BTTvE5tcM0Z5arGmboOold2msYn6TxhN6pDr822QfAIPycd3AJHkpGy35TD9bNBzjevWsq15Hxv6f1mxc2b1l4q/qcNyjvqkRzI2gexVq3ncWuaue3udk2w53ZfhjF9OP7RmR96czd11j7z0@wnPNzOa98WqeR/Ob8fLGd@3e1V3hi5aZUtlnyaix@w2@a3GJuoGdIt5RyiKEU9xAhVr57wjpNu3NV/cYkqzbd7n9oUEzhrOp67D7iLhVt1nS1doOAG@bt@Nh8gO@bycFChwurtiCasV4pqlDoep4Djkrtkz6EvymnZfV@@sxKuL/JIVQC81Bn1pzKf7lWyVPXnytBvFad4kqsroAFB7yqFonor/hOghw1PuDW9C5Mq0@e2TH5qmDkNbJdqXIqWITsb87ON1XxbBrXabeTj94bdGsuZ9nDPbdFI34enL/QpNmC5pyvNvcFhvr6xUOh8@0G2QR3nn9bsfuv@cmklNolBNkTKuJpnXhNdJT0EdMrpymtrTw9qcesHU06aw12@qTJZXxYEETZZwDv80e4OGNNBTt5zMm8znmZRDEaXbS26cJ88/4fhG1MLzOZMxtbrv1eb/AQ)