---
layout: post
title: "NSURLSession的日常"
date: 2014-07-27 13:42:05 +0800
comments: true
categories: 
---
 在使用`NSURLSessionTaskDelegate`的时候，一开始并没有进入`URLSession:task:didCompleteWithError:`回调，阅读苹果给出的文档之后，成功解决。
### NSURLSession简介
 `NSURLSession`是iOS7带来的诸多新特性之一，旨在替换`NSURLConnection`。`NSURLSession`的使用非常简单，步骤如下：

 1.创建并配置`NSURLSessionConfiguration`;

 2.通过步骤一的configuration创建`NSURLSession`;
 
 3.创建相应sessionTask(data、upload or download);

 4.调用`resume`启动task。
 
 给出一些例子代码：
 
    NSData *imageData = UIImageJPEGRepresentation([UIImage imageNamed:@"panda.jpg"], 0.6);

    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    config.HTTPMaximumConnectionsPerHost = 1;
     [config setHTTPAdditionalHeaders:@{@"example_appkey": @"jfdeh79685eun"}];

    NSURLSession *upLoadSession = [NSURLSession sessionWithConfiguration:config];
    NSURL *url = [NSURL URLWithString:@"http://example.com"];
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url];
    [request setHTTPMethod:@"PUT"];

    NSURLSessionUploadTask *uploadTask = [upLoadSession uploadTaskWithRequest:request fromData:imageData completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSHTTPURLResponse *httpResp = (NSHTTPURLResponse*) response;
        if (!error  && httpResp.statusCode == 200) {
            //handle...
        }
    }];

    [[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:YES];
    [_uploadTask resume];
 
## 使用NSURLSessionTaskDelegate
 修改创建uploadSession方法：
    NSURLSession *upLoadSession = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:nil];

 实现`<NSURLSessionTaskDelegate>`，该委托提供出了能计算上传进度的方法，代码如下：

    - (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
    {
        NSLog(@"progress:%f",(double)totalBytesSent / (double)totalBytesExpectedToSend);
    }
 
 将创建uploadTask方法更改为:
    NSURLSessionUploadTask *uploadTask = [upLoadSession uploadTaskWithRequest:request fromData:imageData];

 当然还有sessionTask完成时的回调：

    - (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
    {
        dispatch_async(dispatch_get_main_queue(), ^{
            [[UIApplication sharedApplication] setNetworkActivityIndicatorVisible:NO];
        });
        if (!error) {
            //handle..
        } else {
            // Alert for error
        }
    }

 但是该委托方法没有被调用。苹果文档指出，如果使用自定义的委托来获取数据，那么以下方法必须被实现：
 
 `URLSession:dataTask:didReceiveData:` 一点一点地从你发出地请求中取回数据；
 `URLSession:task:didCompleteWithError:` 数据被完全接收。
 
 由此可以看出，还需要实现`<NSURLSessionDataDelegate>`：
    -(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data
    {
        
    }
 
 至此，成功进入`URLSession:task:didCompleteWithError:`。
 


 