---
title: "Building real-time apps with AWS AppSync Events’ WebSocket publishing"
url: "https://aws.amazon.com/blogs/mobile/building-real-time-apps-with-aws-appsync-events-websocket-publishing/"
date: "Thu, 13 Mar 2025 21:04:20 +0000"
author: "Brice Pellé"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>Real-time features have become essential in modern applications. Whether you’re building collaborative tools, live dashboards, or interactive games, users have come to expect instant and seamless updates as they interact with apps. <a href="https://aws.amazon.com/appsync/">AWS AppSync Events</a>, a fully-managed service for serverless WebSocket APIs, has been helping developers add real-time capabilities to their applications, enabling them to build responsive and engaging experiences at scale.</p> 
<p>Today, I’m excited to announce an enhancement to AWS AppSync Events: the ability to publish messages directly over WebSocket connections, in addition to publishing over the API’s HTTP endpoint. This update allows developers to use a single WebSocket connection for both publishing and receiving events, streamlining the development of real-time features and reducing implementation complexity. This is particularly beneficial for chatty applications that previously faced the overhead of establishing new HTTP connections for each message publication. Developers now have more flexibility: they can publish over HTTP endpoint, e.g.: for publishing from a backend, or publish over WebSocket which might be preferred for web and mobile application clients. In this post, I’ll go over the new enhancement, and show you how you can start integrating publishing over WebSocket in your apps.</p> 
<h2>Getting started</h2> 
<p>As I mentioned in my Announcing AWS AppSync Events <a href="https://aws.amazon.com/blogs/mobile/announcing-aws-appsync-events-serverless-websocket-apis/">post</a>, getting started with AppSync Events is simple, and you can now publish over WebSocket or HTTP directly from the console. From the <a href="https://console.aws.amazon.com/appsync">AppSync console</a>, you can create an API and automatically get a <code>default</code> channel namespace along with an API key. on the Pub/Sub Editor, you can immediately try out your API. As shown in the image below, choose the “Publish” button, and in the dropdown, choose “WebSocket”. Your events are published over the WebSocket. You receive a <code>publish_success</code> message confirming the request.</p> 
<div class="wp-caption aligncenter" id="attachment_13892" style="width: 1447px;">
 <img alt="AppSync Pub/Sub editor showing new &quot;Publish&quot; dropdown button with &quot;HTTP&quot;, and &quot;WebScoket&quot; option" class="wp-image-13892 size-full" height="560" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/appsync-console-publish.png" width="1437" />
 <p class="wp-caption-text" id="caption-attachment-13892">Publishing over WebSocket in the AppSync’s console Pub/Sub editor</p>
</div> 
<h2>Message format</h2> 
<p>AppSync now supports a new “publish” WebSocket operation to publish events. After connecting to the WebSocket, your client can start publishing events to channels in configured channel namespaces. Note that you do not need to subscribe to a channel before publishing to it. To publish events, you simply create a data message, specify an <strong>id</strong> that uniquely identifies the message, the <strong>channel</strong> you are sending the message to, the list of <strong>events</strong> your are sending (up to 5), and the <strong>authorization</strong> headers to allow the request. You can find more information about the authorization headers format in the <a href="https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-websocket-protocol.html#authorization-formatting-by-mode">documentation</a>. Each event in the events array must be a valid JSON string. Here’s an example.</p> 
<pre><code class="language-json">{
  "type": "publish",
  "id": "an-identifier-for-this-request",
  "channel": "/namespace/my/path",
  "events": [ "{ \"msg\": \"Hello World!\" }" ],
  "authorization": {
    "x-api-key": "da2-12345678901234567890123456-example",
   }
}
</code></pre> 
<p>After publishing, you receive a “publish_success” response with details on each event sent, or a “publish_error” response if the operation was not successful.</p> 
<h2>Integrate with your application</h2> 
<p>In the previous post, I showed you how to implement a simple client that uses the browser’s Web API <a href="https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API">WebSocket</a>. I’ll do this again to connect and publish on an Event API WebSocket endpoint. In this example, I’ll build a real-time demo chat application where clients can send and receive messages instantly over WebSocket. Messages are published to the <code>/default/messages</code> channel, and clients subscribe to <code>/default/*</code> to receive all messages in the default namespace.</p> 
<div class="wp-caption aligncenter" id="attachment_13900" style="width: 749px;">
 <img alt="A simple chat titled &quot;Messages&quot; show some received messages above an input form" class="wp-image-13900 size-full" height="367" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/pub-sub-messages.png" width="739" />
 <p class="wp-caption-text" id="caption-attachment-13900">A demo chat application</p>
</div> 
<p>I’ll use the new <a href="https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_appsync-readme.html#events">AWS Cloud Development Kit (CDK) L2 constructs for AppSync Events</a> to configure and deploy an AppSync Event API, with a single channel namespace named “default”, and a single API key. <a href="https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html">Learn more</a> about getting started with AWS CDK. If needed, use the Node Package Manager to install the CDK CLI.</p> 
<pre><code class="language-sh">$ npm install -g aws-cdk
</code></pre> 
<p>I start by initializing a new folder structure and create my CDK application.</p> 
<pre><code class="language-sh">$ mkdir -p events-app/cdk-events-publish
$ cd events-app/cdk-events-publish
$ cdk init app --language javascript
</code></pre> 
<p>Next, I update the <code>lib/cdk-events-publish-stack.js</code> file with this code:</p> 
<pre><code class="language-js">const { Stack, CfnOutput } = require('aws-cdk-lib');
const { EventApi, AppSyncAuthorizationType } = require('aws-cdk-lib/aws-appsync');

class CdkEventsPublishStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);
    const apiKeyProvider = { authorizationType: AppSyncAuthorizationType.API_KEY };

    // create an API called `my-event-api` that uses API Key authorization
    const api = new EventApi(this, 'api', {
      apiName: 'my-event-api',
      authorizationConfig: { authProviders: [apiKeyProvider] }
    });

    // add a channel namespace called `default`
    api.addChannelNamespace('default');

    // output configuration properties
    new CfnOutput(this, 'apiKey', { value: api.apiKeys['Default'].attrApiKey });
    new CfnOutput(this, 'httpDomain', { value: api.httpDns });
    new CfnOutput(this, 'realtimeDomain', { value: api.realtimeDns });
  }
}

module.exports = { CdkEventsPublishStack }
</code></pre> 
<p>I then deploy the stack and save the output to a JSON file.</p> 
<pre><code class="language-sh">$ npm run cdk deploy -- -O output.json
</code></pre> 
<p>I get output that looks like this:</p> 
<pre><code class="language-txt">Outputs:
CdkStack.apiKey = da2-12345678901234567890123456-example
CdkStack.httpDomain = a12345678901234567890123456.appsync-api.us-east-2.amazonaws.com
CdkStack.realtimeDomain = a12345678901234567890123456.appsync-realtime-api.us-east-2.amazonaws.com
</code></pre> 
<p>Next, I create the web app using <a href="https://vite.dev/">vite</a> (a frontend build tool), and vite’s vanilla javascript template. From the <code>events-app</code> folder, I run:</p> 
<pre><code class="language-sh">$ npm create vite@latest app -- --template vanilla
$ cd app
$ npm install
$ ln -s ../../cdk-events-publish/output.json src
</code></pre> 
<p>This creates a link to the <code>output.json</code> file that I can refer to in the app. In the new <code>app</code> folder, I replace <code>src/main.js</code> with the code below. Note that the code uses the CDK output in <code>output.json</code>.</p> 
<pre><code class="language-js">import './style.css'
import output from './output.json'

// use the output from the CDK stack deployment
const HTTP_DOMAIN = output.CdkEventsPublishStack.httpDomain
const REALTIME_DOMAIN = output.CdkEventsPublishStack.realtimeDomain
const API_KEY = output.CdkEventsPublishStack.apiKey 
     
const authorization = { 'x-api-key': API_KEY, host: HTTP_DOMAIN }

document.querySelector('#app').innerHTML = `
    &lt;style&gt;
      #top{width: calc(100vw - 8rem); max-width: 1280px; background: oklch(0.977 0.013 236.62); margin: 2rem 0; height: calc(100vh - 8rem); box-shadow: inset 0 0 20px rgba(0,0,0,0.1); position: relative; margin-inline: auto;}
      #container{display: flex;flex-direction: column;height: calc(100% - 6.5rem);}
      h2{color: oklch(0.3 0.013 236.62); margin-bottom: 1.5em; font-size: 1.8rem;}
      #messages{display: flex; flex-direction: column-reverse; gap: 1em; max-height: calc(100vh - 200px); flex: 1; min-height: 0; overflow-y: auto; text-align: left;}
      form{position: absolute; bottom: 2rem; left: 2rem; right: 2rem; padding: 1rem 0;}
      input{width: 90%; padding: 0.8em 1.2em; border: 1px solid oklch(0.8 0.013 236.62); border-radius: 25px; font-size: 1rem; outline: none; transition: all 0.2s ease; box-shadow: 0 2px 5px rgba(0,0,0,0.05);}
      .msg{padding: 0 1rem; font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace;}
    &lt;/style&gt;
    &lt;div id="top"&gt;
      &lt;div id="container"&gt;&lt;h2&gt;Messages&lt;/h2&gt;&lt;div id="messages"&gt;&lt;/div&gt;&lt;/div&gt;
      &lt;form id="form"&gt; &lt;input id="messageInput" name="message" type="text" autocomplete="off" /&gt;&lt;/form&gt;
    &lt;/div&gt;`

// construct the protocol header for the connection
function getAuthProtocol() {
  const header = btoa(JSON.stringify(authorization))
    .replace(/\+/g, '-') // Convert '+' to '-'
    .replace(/\//g, '_') // Convert '/' to '_'
    .replace(/=+$/, '') // Remove padding `=`
  return `header-${header}`
}

const socket = await new Promise((resolve, reject) =&gt; {
  const socket = new WebSocket(`wss://${REALTIME_DOMAIN}/event/realtime`, [
    'aws-appsync-event-ws',
    getAuthProtocol(),
  ])
  socket.onopen = () =&gt; resolve(socket)
  socket.onclose = (event) =&gt; reject(new Error(event.reason))
  socket.onmessage = (_evt) =&gt; {
    const data = JSON.parse(_evt.data)
    // if this is a `data` event, add the content to the list of messages
    if (data.type === 'data') {
      const event = JSON.parse(data.event)
      const div = document.createElement('div')
      div.className = 'msg'
      div.innerHTML = `↑ ${event.time} | ↓ ${new Date().toISOString().split('T')[1]} | ${event.message}`
      messages.prepend(div)
    }
  }
})

// subscribe to `/default/*`
socket.send(
  JSON.stringify({
    type: 'subscribe',
    id: crypto.randomUUID(),
    channel: '/default/*',
    authorization,
  }),
)

const form = document.querySelector('#form')
const messageInput = document.querySelector('#messageInput')
const messages = document.querySelector('#messages')

// when the form is submitted, send an event to `/default/messages`
form.addEventListener('submit', (e) =&gt; {
  e.preventDefault()
  const message = new FormData(e.currentTarget).get('message')
  messageInput.value = ''
  socket.send(
    JSON.stringify({
      type: 'publish',
      id: crypto.randomUUID(),
      channel: '/default/messages',
      events: [JSON.stringify({ message, time: new Date().toISOString().split('T')[1] })],
      authorization,
    }),
  )
})
</code></pre> 
<p>Now in the <code>app</code> directory, I start the web server:</p> 
<pre><code class="language-sh">$ npm run dev
</code></pre> 
<p>I can open the website at the provided address on multiple browsers to send and receive messages.</p> 
<h3>Cleaning up</h3> 
<p>When done with the example, I can remove the resources I deployed with CDK.</p> 
<pre><code class="language-sh">$ cd ../cdk-events-publish
$ npm run cdk destroy
</code></pre> 
<h2>Conclusion</h2> 
<p>In this post, I went over the new Publish over WebSocket feature for AWS AppSync Events, and showed how to get started with the feature. This enhancement offers several benefits:</p> 
<ul> 
 <li>Simplified implementation with a single connection for publishing and subscribing</li> 
 <li>Reduced connection overhead for chatty applications</li> 
 <li>Flexibility to choose between WebSocket and HTTP publishing based on your use case</li> 
</ul> 
<p>Publishing over WebSocket is now available in all regions where AppSync is available. Clients can publish at a rate of 25 requests per second per client WebSocket connection. You can continue using your API’s HTTP endpoint to publish at higher rates (adjustable default of 10,000 events per second). Visit AppSync’s <a href="https://docs.aws.amazon.com/general/latest/gr/appsync.html">endpoints and quotas page</a> for more details. We’re excited to see what you build with this new capability. Get started today by trying out the example in this post or by adding WebSocket publishing to your existing AppSync Events applications. To learn more about AppSync Events, visit the <a href="https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-welcome.html">documentation</a>.</p>
