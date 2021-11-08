# OHAI Working Group Agenda - IETF 112

### Overview of [draft-thomson-http-oblivious](https://datatracker.ietf.org/doc/draft-thomson-ohai-ohttp/) (Chris Wood)

**Chris Wood**: Cases where interacting with a server reveals sensitive information. Interaction with server also reveals identity. Examples: DNS queries, Safe Browsing, telemetry data uploads

**Chris**: Many applications use HTTP for _transactional_ tasks. Existing solutions trade off linkability for connection setup overhead.

**Chris**: Oblivious HTTP splits identity from the request iteslf by using a proxy. Proxy knows identity, target knows request. Encrypt request/response with keys from target so proxy cannot see them. Requires that proxy doesn't reveal identity to target. Target trusts proxy to perform DoS protection.

**Chris**: Not a general purpose proxying protocol. Discovery out of scope. 

**Chris**: At least 2 interoperable implementations so far. 

**Tommy Pauly**: I did recently implement both this and binary HTTP messages. Both are good and ready for adoption. Clarifying questions: One thing that wasn't obvious in document, but was in implementation was that the proxy needs a fairly static mapping for when you give it a request, requires state. Binary HTTP being the body of this is pretty disconnected. In ODoH, we didn't have the interior of the oblivious message be HTTP, it was fine but do we need to have it specifying that? Couldn't both the client and the target decide to put whatever they want in there? Oblivious bodies over HTTP, we don't need to care what's in them.

**Chris**: The latter, as you point out we're only really focusing on encapsulation and decapsulation. No real requirement as to what's inside. As to configuration, we can include more information about how to handle that and how to sort out clients with requests for particular targets. Please file an issue.

**Richard Barnes**: Note from the chairs, there's been a bunch of conversation about collusion and proxy selection in the chat. From the BoF and charter, we're assuming that you've already selected a proxy and target server that it assumes are not colluding, anything further is out of scope for this working group. The protocol's job is to make sure that things are verifiable and authenticatable by the client, not to prevent collusion.

**Andrew Campling**: I think that's huge, we need to make sure that we're not just providing the illusion of privacy for the end user.

**Chris**: Agreed, but for now we're operating under the assumption that the configuration has already happened.

**Stephen Farrell**: I'm not asking for discovery, I'm asking for discovery of collusion. Is there any sort of way that we could make possible some kind of transparency? If this protocol provides no mechanism by which collusion can be detected, even if later, then the overall benefit is harder to see given who is likely to put up these boxes.

**Chris**: Not sure this is significantly different.

**Stephen**: Certificate transparency-style thing perhaps?

**Richard**: If there are concrete proposals, please propose to the list, but I think it's clear from the charter that this is not a blocker for adoption.

### Overview of [draft-thomson-http-binary-message](https://datatracker.ietf.org/doc/draft-thomson-http-binary-message/) (Martin Thomson)

**Martin Thomson**: At the moment, OHTTP uses binary HTTP, filed an issue for Tommy's question about carrying any payload. Everything is included in a binary encoding which is unambiguous and easy to parse.

**Martin**: It's not quite HTTP/2 or HTTP/3, those are fairly complicated. No HPACK (unclear if compression is safe, and may scare people away). Two types: fixed length and indeterminate length. Depending on if you know length or not, you may be getting a message from somewhere so use whichever fits your situation.

**Martin**: Probably not something for OHAI to do, suggesting that we ask HTTP to do it.

**Richard**: Is OHAI useful without this?

**Martin**: Depends on how much we rely on Tommy's idea that it can carry anything. I think this is the right thing to do in terms of carrying HTTP, so I'd prefer that we do this.

**Richard**: Seems like we at least need a request/response semantic at minimum, regardless of what's being carried.

**Martin**: Correct, that's a limiation we need to discuss for HTTP as well. For later, but do we want to have any kind of streaming capability?

**Eric Rescorla**: Seems like a decent starting point, I don't care if it carries binary HTTP or something else. Richard, thanks for trying to scope this as narrowly as possible. Let's go adoption call.

**Watson Ladd**: In terms of drafts and WGs, more people who know about HTTP in the HTTP group than in this group. 

**Martin**: There are some fiddly bits in here, worth getting HTTP to do that work.

**Richard**: Even if we keep this group independent of HTTP bits, we can still take this work to them and have that dependency.

### Adoption call (Chairs)

**Richard**: Raise hand for good starting point. Click do not raise hand if you have concerns and believe it is not a good starting point.

134 people in room, 61 participated, 51 raised hands, 10 did not raise hand

![](https://notes.ietf.org/uploads/upload_d86127ab1fc265e39c451190e1aba806.png)

**Richard**: Will confirm list, sounds like we've got pretty good consensus to adopt this and continue work as a working group item.

### Issue discussion (Martin Thomson)

[Issue List](https://github.com/unicorn-wg/oblivious-http/issues/)

#### [Streaming Request/Response](https://github.com/unicorn-wg/oblivious-http/issues/75)

**Martin**: You could have longer bodies, things that are processed incrementally. Cost is primarily some sort of framing and a little bit of framing.

**Ted Hardie**: If I currently have a setup where I have three of these proxies configured, one thing I can do is send information to one/two/three, round robin to same target via different proxies. That works fine because they're for short transactions that are essentially stateless. If you made them more generic, then don't you need some more generic way of thinking about the interaction set? 

**Martin**: I don't think we're looking to change the interaction model, you keep the same capabilities. Difference is that you can construct request/response such that pieces of it will be available for processing rather than the entire thing.

**Ted**: So you would still say that if I wanted to switch to the next proxy in my list, it's just that the first response might keep continuing (like with a 100)?

**Martin**: Yes **Ted**: Thanks

**Tommy**: I'd like to understand what specific applications would need this more generic format. I've been thinking that we have oblivious requests for the short transactions and we use multi-hop MASQUE proxies for more stateful items. Can we get away with just that?

**Martin**: I think that's reasonable as far as approach, that's what I did originally, but important to ask.

**Richard**: The choice here is between flexibility and having a simpler interaction model? Declare that some HTTP patterns cannot be done over this channel?

**Martin**: Nothing here says that we cannot define a new format to do more complicated things later if we need, but for now keeping the overhead low and keeping it as simple as possible. As Tommy says, because of the other options we can keep that out of scope and keep this targeted.

**Jonathan Hoyland**: Does this create a vector whereby you can more easily make a server store a lot of sate by sending many "part 1 of 6" messages? 

**Martin**: I think that was true already, but it shifts where you maintain that state. In current design, you've sent the same thing but server cannot decrypt and is now holding ciphertext. This change would allow decrypting that and shift holding that state to the application layer.

**Mark Nottingham**: I was asking some of these questions, I don't have direct strong feelings about it, but this is one of the things where it felt like we often put these artificial barriers in place that applications then immediately trip over. Also naming issue, if we're not doing full HTTP then we probably shouldn't call it anything-dash-HTTP, but that's a bikeshed for later. There are some examples of applications that are going to want this pretty quickly.

**Martin**: I made the observation that we can make new media types in order to support different use cases.

**Mark**: As I understand it, this is a question for a generic proxying layer/how you bundle up the crypto, but also a question for 1xx about the HTTP format.

**Martin**: Good question, I don't have text to generate live for that right now.

**Mark**: Fine if we decide to keep it limited, but if we do that should be a deliberate choice.

#### [Additional Data](https://github.com/unicorn-wg/oblivious-http/issues/70)

**Martin**: Is there anything that we should expose to the intermediary for it to more effectively do its job. My initial take is nothing at all. Not sure we need to discuss here, but worth thinking about for later. Another point is anti-replay capabilities. Proxy becomes a source of replay attacks against the server, providing some sort of facilities for preventing that is not included, but we might want to do something like put timestamps in messages or something else to enable some additional anti-replay bits.

**Chris**: For additional context, over in DTN we're reusing HPKE for encrypting data that fits inside of an HTTP request from clients to these servers. It's been discussed that we might want to use OHAI/OHTTP for sending those messages, but those need to send some parts in the clear that's still authenticated. Could use reuse this for that. I do tend to agree with the pushback that we don't want to add more things and additional channels for the proxy.

### Virtual interims

**Richard**: Good meeting so far, clear adoption call which we'll confirm on the list. As we're focused on burning down issues, do we want to schedule some virtual interims scheduled to go through issues and keep up the pace? Thoughts?

**Martin**: I have a preference for doing things in GitHub, mailing list is fine to be able to make async progress online for now.

**Richard**: Sounds good, 2 weeks notice if we ever want to have some face to face time.
