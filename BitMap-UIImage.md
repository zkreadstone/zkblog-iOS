IOS开发中位图BitMap的使用
在子线程中先将图片绘制到CGBitmapContext，然后从Bitmap 直接创建图片，这样做的目的减轻主线程的压力，图片解码等耗性能操作放到异步子线程解码。
解析图片根据BitMap生成图片，提高图片加载效率

```Objective-c
dispatch_async(queue, ^{
 CGImageRef cgImage = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:self]]].CGImage;
 CGImageAlphaInfo alphaInfo = CGImageGetAlphaInfo(cgImage) & kCGBitmapAlphaInfoMask;
 
 BOOL hasAlpha = NO;
 if (alphaInfo == kCGImageAlphaPremultipliedLast ||
     alphaInfo == kCGImageAlphaPremultipliedFirst ||
     alphaInfo == kCGImageAlphaLast ||
     alphaInfo == kCGImageAlphaFirst) {
     hasAlpha = YES;
 }
 
 CGBitmapInfo bitmapInfo = kCGBitmapByteOrder32Host;
 bitmapInfo |= hasAlpha ? kCGImageAlphaPremultipliedFirst : kCGImageAlphaNoneSkipFirst;
 
 size_t width = CGImageGetWidth(cgImage);
 size_t height = CGImageGetHeight(cgImage);
 
 CGContextRef context = CGBitmapContextCreate(NULL, width, height, 8, 0, CGColorSpaceCreateDeviceRGB(), bitmapInfo);
 CGContextDrawImage(context, CGRectMake(0, 0, width, height), cgImage);
 cgImage = CGBitmapContextCreateImage(context);
 
 UIImage * image = [[UIImage imageWithCGImage:cgImage] cornerRadius:width * 0.5];
 CGContextRelease(context);
 CGImageRelease(cgImage);
 completion(image);
});
```
