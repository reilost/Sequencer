Sequencer
=========

Sequencer is an iOS library for asynchronous flow control.

Sequencer turns complicated nested blocks logic into a clean, straightforward, and readable code.

## Jumping straight to code

Let's output some strings to the Log Console:

```objc
Sequencer *sequencer = [[Sequencer alloc] init];
[sequencer enqueueStep:^(id result, SequencerCompletion completion) {
    NSLog(@"This is the first step");
    completion(nil);
}];
[sequencer enqueueStep:^(id result, SequencerCompletion completion) {
    NSLog(@"This is another step");
    completion(nil);
}];
[sequencer enqueueStep:^(id result, SequencerCompletion completion) {
    NSLog(@"This step is going to do some async work...");
    int64_t delayInSeconds = 2.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);
    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
        NSLog(@"finished the async work.");
        completion(nil);
    });
}];
[sequencer enqueueStep:^(id result, SequencerCompletion completion) {
    NSLog(@"This is the last step");
    completion(nil);
}];
[sequencer run];
```

What does the above code do?

1. A ```Sequencer``` was created. There is no need to retain/hold-on-to-it. __Trust me.__
2. Four steps were enqueued to the Sequencer. The third step is async, but all the rest are plain sync code.
3. Each step finishes by calling ```completion()``` with a ```result``` object. This result is sent to the next step (in our case the result is ```nil```).
5. We ```run``` the sequencer.

__Note__: Break the steps by just removing the call to ```completion(nil)```. Everything will be cleaned-up auto-magically.

## A Bettter Example

Let's do something more useful.

We will call the [app.net](https://join.app.net/) API to get the latest feed, read the last feed-item and get its URL and then read the HTML content of the URL.

(Using [AFNetworking](https://github.com/AFNetworking/AFNetworking) for all the networking stuff)

```objc
Sequencer *sequencer = [[Sequencer alloc] init];
[sequencer enqueueStep:^(id result, SequencerCompletion completion) {
    NSURL *url = [NSURL URLWithString:@"https://alpha-api.app.net/stream/0/posts/stream/global"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    AFJSONRequestOperation *operation = [AFJSONRequestOperation JSONRequestOperationWithRequest:request success:^(NSURLRequest *request, NSHTTPURLResponse *response, id JSON) {
        completion(JSON);
    } failure:nil];
    [operation start];
}];
[sequencer enqueueStep:^(NSDictionary *feed, SequencerCompletion completion) {
    NSArray *data = [feed objectForKey:@"data"];
    NSDictionary *lastFeedItem = [data lastObject];
    NSString *cononicalURL = [lastFeedItem objectForKey:@"canonical_url"];

    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:cononicalURL]];
    AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
        completion(responseObject);
    } failure:nil];
    [operation start];
}];
[sequencer enqueueStep:^(NSData *htmlData, SequencerCompletion completion) {
    NSString *html = [[NSString alloc] initWithData:htmlData encoding:NSUTF8StringEncoding];
    NSLog(@"HTML Page: %@", html);
    completion(nil);
}];
[sequencer run];
```

1. The first step downloads a JSON object, turns it into an NSDictionary and sends it to the next step.
2. The second step reads the canonicalURL, downloads it and send it to the next step.
3. The third and last step turns the data into a string and outputs it.

## Contact

Find me on github: [Tal Bereznitskey](http://github.com/berzniz)

Follow me on Twitter: [@ketacode](https://twitter.com/ketacode)
