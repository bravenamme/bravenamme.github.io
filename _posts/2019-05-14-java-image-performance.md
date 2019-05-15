---
layout: post
title:  "Java Image Performance"
date:   2019-05-14 10:00
author: twosunny
tags:	[java,bufferedImage,pngEncoder]
---

실제 업무에서 Java로 image를 처리하여 서비스 하는 경우는 그렇게 많지 않을것이다.  
예시 프로젝트를 통해, Java로 이미지를 읽고, 가공하고, 다시 인코딩을 하는 일련의 과정을 쓸 예정이다.  
하지만 이미지 처리에서 가장 중요한 건 역시 **속도**다. 멋있는 이미지를 서버에서 만들었다 해도, 다수의 접속자가  
쾌적하게 그 이미지를 볼수 있어야 한다.

그런 의미에서 **performance** 중심 위주로 기술하려고 한다. 



## 예시 프로젝트 소개 

아래와 같은 1~9까지의 이미지 조각들을 읽어서(image decoding),

<img src="/files/posts/1.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/2.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/3.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/4.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/5.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/6.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/7.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/8.png" style="width:10%;height:10%;float:left;">
<img src="/files/posts/9.png" style="width:10%;height:10%;float:left;"><br><br><br>


    
각 조각 이미지를 `BufferedImage`를 이용하여 합친뒤, 합친 이미지를 아래와 같이 png로 변환(image encoding)  
할 것이며, 각 단계에 따른 속도 측정을 할 예정이다.(stress tool은 [ngrinder](http://naver.github.io/ngrinder/)를 사용 )
![합친 이미지](/files/posts/combineImages.png)

예시에 사용된 모든 소스는 [여기][example]에 가면 볼수 있다

### 이미지 로딩

숫자 9개의 이미지를 모두 로딩해 보자.
[ImageIO][ImageIO]를 사용하여 아래 3가지 방법으로 이미지를 로딩 후 BufferedImage로 변환 한다.
* 단순 file path 로 로딩  
```java
public BufferedImage getImage(String filePath) throws IOException{
	BufferedImage image = null;	
	image = ImageIO.read(new File(filePath));
	return image;
}
```
성능 측정 결과

![](/files/posts/read_normal.png)

* [Java NIO](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html) 로 로딩  
```java
public BufferedImage getImageByNio(String filePath) throws IOException {
		
	BufferedImage image = null;
	
	Path path = Paths.get(filePath);
	ByteArrayInputStream bais  = null;
	
	try {
		if(Files.exists(path)) {
			byte[] imageBytes = Files.readAllBytes(path);
			bais = new ByteArrayInputStream(imageBytes);
			image = ImageIO.read(bais);
		}
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if(bais != null) bais.close();
	}
	
	return image;
}
```
성능 측정 결과

![](/files/posts/read_nio.png)

* BufferedInputStream 로 로딩  
```java
public BufferedImage getImageByBuffered(String filePath) throws IOException {
	
	BufferedImage image = null;
	FileInputStream fis = null;
	BufferedInputStream bis = null;
	
	try {
		fis = new FileInputStream(new File(filePath));
		bis = new BufferedInputStream(fis);
		image = ImageIO.read(bis);
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if(bis != null) bis.close();
		if(fis != null) fis.close();
	}
	
	return image;
}
```
성능 측정 결과

![](/files/posts/read_buffered.png)

  
약간의 차이가 있긴 하지만, 어떤 방식이던간에 이미지 데이터 자체를 로딩하고, [ImageIO][ImageIO]를 통해 `BufferedImage`를 만드는 속도 차이는 피부로 느낄만한 수준이 아니다.

### 이미지 합치기 및 이미지 인코딩

 간단한 restful api 를 만들어 api를 호출하면 이미 로딩된 1~9까지의 이미지를 합치고, png로 인코딩후, png	자체를  
 response로 리턴하게 한다.
 
 api 소스는 아래와 같다.
```java
@RequestMapping("/combineImages")
public ResponseEntity<byte[]> combineNumbers(HttpServletRequest request,
		HttpServletResponse response){
	HttpHeaders headers = new HttpHeaders();
	headers.setContentType(MediaType.IMAGE_PNG);
	
	ImageWriterType writerType = null;
	String write = request.getParameter("write");
	
	if(StringUtils.isEmpty(write) || write.equals("1")) {
		writerType = ImageWriterType.ImageIO;
	}else if(write.equals("2")) {
		writerType = ImageWriterType.JDELI;
	}else {
		writerType = ImageWriterType.OBJECTPLANET;
	}
	
	BufferedImage image = new BufferedImage(150, 150, BufferedImage.TYPE_INT_ARGB);
	Graphics2D graphics = image.createGraphics();
	
	int x = 0;
	int y = 0;
	
	for(int i=1;i<10;i++) {
		
		BufferedImage numberImage = imageMap.get(i);
		graphics.drawImage(numberImage, x, y, null);
		
		x += 50;
		if(i%3 == 0) {
			x = 0;
			y += 50;
		}
	}
	
	graphics.dispose();
	
	ByteArrayOutputStream baos = new ByteArrayOutputStream();
	imageService.writeImage(writerType, baos, image);
	
	return new ResponseEntity<byte[]>(baos.toByteArray(), headers, HttpStatus.OK);
}
```
 
 먼저 150X150 의 `BufferedImage`를 생성후, 이미 로딩된 1~9까지의 이미지를 합치고, 그 다음  
 png로 인코딩 후, response로 던져주는 소스이다.
 
 합쳐진 이미지에 대한 png enconding은 아래와 같은 3가지 방식으로 처리 했다.
 
 * [ImageIO][ImageIO] 인코딩
 ```java
public void writeImage(OutputStream outputStream, BufferedImage image) throws IOException{
	try {
		ImageIO.write(image, "png", outputStream);
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if(outputStream != null) outputStream.close();
	}
}
```

성능 측정 결과

![](/files/posts/write_imageio.png)

 * [JDeli PngEncoder](https://www.idrsolutions.com/jdeli)
 ```java
public void writeImage(OutputStream outputStream, BufferedImage image) throws IOException {
	
	try {
		
		PngEncoder encoder = new PngEncoder();
		
		encoder.setCompressed(true);
		encoder.write(image, outputStream);
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if(outputStream != null) outputStream.close();

	}
}
```

성능 측정 결과

![](/files/posts/write_jdeli.png)

 * [objectplanet PngEncoder](http://objectplanet.com/pngencoder/)
 ```java
public void writeImage(OutputStream outputStream, BufferedImage image) throws IOException {
	
	try {
		
		PngEncoder encoder = new PngEncoder();
		
		encoder.setColorType(PngEncoder.COLOR_TRUECOLOR_ALPHA);
		encoder.setCompression(PngEncoder.DEFAULT_COMPRESSION);
		
		encoder.encode(image, outputStream);
	}catch(Exception e) {
		e.printStackTrace();
	}finally {
		if(outputStream != null) outputStream.close();

	}
}
```

성능 측정 결과

![](/files/posts/write_object.png)

`BufferedImage`를 png로 인코딩 시키는데 있어서, 어떠한 라이브러리를 쓰는지에 따라서 **굉장한 차이**가 나는것을 볼수 있다. 적어도 Java에서 image를 생성하여 실시간 서비스를 하기 위해서는, [ImageIO][ImageIO]로 png를 생성하는 부분은 피하는게 
좋을것 같다.

### 마치며

Java image 처리는 위에 글 외에, `blending`, `composite`, `xBRZ` 등 여러 이미지 가공이 가능하다.  
대용량 트래픽에서 실시간으로 이미지를 읽고, 디코딩하고, 합성하고, 다시 인코딩 하는 처리는 부담스러울수   밖에 없다. 특히 java에서 image를 인코딩, 디코딩 하는 부분은 예상외로 시간이 좀 걸린다. 
 
소스단에서 이미지 처리 프로세스를 하나하나씩 점검하면서, 계속 튜닝을 해야할 것이다. 또한 File I/O 가   빈번히 일어나며, 이미지 처리 특성상 cpu도 많이 사용함으로, 하드웨어적으로는 SSD 및 고사양 CPU가 받쳐준다면, 유저 서비스하는데 큰 무리는 없을 것이다.

예제에 사용된 모든 소스는 [여기][example]에서 볼 수 있다.

[ImageIO]: https://docs.oracle.com/javase/8/docs/api/javax/imageio/ImageIO.html
[example]: https://github.com/twosunny/ts-java-images

