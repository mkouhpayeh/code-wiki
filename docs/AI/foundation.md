
# AI Foundation

## Simple Chat 


## Streaming
When we compare our chat application with the official ChatGPT interface, there is a big difference. Even though the generation speed was pretty much the same, our app still felt a lot slower. Why? </br>
Because our app waited until the entire poem was ready before showing it. And ChatGPT showed the text as it was being written word by word. This is called **streaming**. ChatGPT displays the response in chunks, allowing us to start reading the answer much earlier. </br>
The current generation speed of GPT-4 is slightly faster than what a human can read. This means that if we stream the response, the user can essentially follow along and doesn't perceive the AI as slow. </br></br>

As cool as streaming is, it has its disadvantages:

- It's more complex to implement. Instead of simply receiving and displaying a complete response

- We have to handle partial responses, keep track of the chunks, handle intermediate errors, and so on

- Requires more bandwidth as every chunk comes with quite a lot of overhead and only one or two useful words worth of content. 
    
Should we use streaming in your project? The answer, as it's pretty much always is, it depends. 

- The model

- The load on the server

- he amount of text you need to generate

- User's expectations or mindset.
