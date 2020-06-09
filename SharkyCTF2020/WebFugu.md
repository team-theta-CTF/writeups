# WebFugu Writeup
>A new site listing the different species of fugu fish has appeared on the net. Used by many researchers, it is nevertheless vulnerable. Find the vulnerability and exploit it to recover some of the website configuration.
>Creator: Masterfox

*Writeup by: @R4H33M*

Going to the website shows us a very fancy list of different types of [fugu](https://en.wikipedia.org/wiki/Fugu), also known as pufferfish.

Always, with web challenges, begin by checking the client-side html source. If you are on Chrome, simply right click and press "View page source". We notice something at the bottom of the source: 

    <!-- Loading fish list script -->
    <script src="/js/script.js"></script>

This script goes like this:

    async function populateList() {
    	let b64ListContent = "PGRpdj4gPHRhYmxlIGJvcmRlcj0iMSI%2BIDx0cj4gPHRoPk5hbWU8L3RoPiA8dGg%2BRGlzY292ZXJ5IHllYXI8L3RoPiA8dGg%2BRGlzY292ZXJlciBuYW1lPC90aD4gPC90cj4gPHRyIHRoOmVhY2ggPSJmaXNoIDogJHtmaXNoZXN9Ij4gPHRkIHRoOnV0ZXh0PSIke2Zpc2gubmFtZX0iPi4uLjwvdGQ%2BIDx0ZCB0aDp1dGV4dD0iJHtmaXNoLmRpc2NvdmVyeVllYXJ9Ij4uLi48L3RkPiA8dGQgdGg6dXRleHQ9IiR7ZmlzaC5kaXNjb3ZlcmVyTmFtZX0iPi4uLjwvdGQ%2BIDwvdHI%2BIDwvdGFibGU%2BIDwvZGl2PiAgICAgIAo=";
	    let renderURL = "/process";
	    let listContent = await fetch(location.origin + renderURL + "?page=" + b64ListContent).then(res => res.text());
	    fishList.innerHTML = listContent;
    }

    document.addEventListener('DOMContentLoaded', function() {
    	var popup = M.Modal.init(addFish);
    });
    
    populateList();

It seems that the list of pufferfish is retrieved through a GET request to "/process", with the parameter "page=". What is filled into this parameter is base64 encoded. How do we know this? 2 reasons: it ends with an enter sign (=) and the variable name is "**b64**ListContent". When I tried to decode it, I got a sliver of something interesting, but the rest of gibberish. What I didn't know was that base64 only uses 64 characters... (shocking, right?). These are A-Z, a-z, 0-9, "+" and "/" (additionally "=" for padding).

As you see, the base64 string we are given also includes the sign "%". hmmmm. Taking into account that the string is appended onto a web address, we recogize that this is simply a [url encoding](https://www.w3schools.com/tags/ref_urlencode.ASP). The "%2F" corresponds to "/" and "%2B" corresponds to "+". Substituting these, and base 64 decoding the string, we get:

    <div> <table border="1"> <tr> <th>Name</th> <th>Discovery year</th> <th>Discoverer name</th> </tr> <tr th:each ="fish : ${fishes}"> <td th:utext="${fish.name}">...</td> <td th:utext="${fish.discoveryYear}">...</td> <td th:utext="${fish.discovererName}">...</td> </tr> </table> </div
    
Fishy! (pun intended) This looks very much like html. But there is something wrong... what is "th:each" and "th:utext"?

Searching them up, we get [thymeleaf](https://www.thymeleaf.org/), a server side Java template engine. Thymeleaf takes templates like the above and serves the information we want, like the fishes and their attributes. Let's see, is the flag a variable? Let's try encoding the following HTML in base 64, and send that as the "page=" parameter instead:

    <p th:utext="${flag}"></p>

We get "Flag [content=REDACTED_IT_WONT_BE_THAT_EASY]".

I ended up getting stuck for a few hours after this, but eventually I saw the similarity between the message we got and a (Java) hashmap. Trying this out:

    <p th:utext="${flag.content}"></p>

We get **shkCTF{w31rd_fishes_6ee9f7aabf6689c6684ad9fd9ffbae5a}**

And that's it! Thanks for reading! :)