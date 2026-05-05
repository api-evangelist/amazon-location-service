---
title: "Simplifying Multi-App Management in AWS Amplify Hosting"
url: "https://aws.amazon.com/blogs/mobile/simplifying-multi-app-management-in-aws-amplify-hosting/"
date: "Mon, 10 Mar 2025 20:09:13 +0000"
author: "Matt Auerbach"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p><a href="https://aws.amazon.com/amplify/hosting/">AWS Amplify Hosting</a> has now made it possible to connect many more Amplify apps to a single repository. This change improves the way developers can integrate with Git providers which is especially beneficial for monorepo architectures. Amplify now uses a single webhook per repository for all associated apps, streamlining the development workflow. For specific limit details, please refer to our <a href="https://docs.aws.amazon.com/amplify/latest/userguide/quotas-chapter.html">documentation</a>.</p> 
<p>Previously, Amplify users were constrained by the webhook limits provided by Git providers. As Amplify created a new webhook for each app associated with a repository, users with multiple Amplify apps linked to a single repository could reach these limits quickily, preventing them from adding more apps. This was particularly challenging for teams working with monorepos, where multiple projects (and thus, multiple Amplify apps) exist within a single repository.</p> 
<p>While the specific number of webhooks allowed varies by Git provider, these limits presented an obstacle for teams looking to scale their projects or work with complex repository structures.</p> 
<ul> 
 <li>GitHub has a webhook <a href="https://docs.github.com/en/webhooks/testing-and-troubleshooting-webhooks/troubleshooting-webhooks#cannot-have-more-than-20-webhooks">limit</a> of 20</li> 
 <li>GitLab has webhook <a href="https://docs.gitlab.com/ee/user/gitlab_com/index.html#other-limits">limit</a> of 100</li> 
 <li>BitBucket has webhook <a href="https://support.atlassian.com/bitbucket-cloud/docs/manage-webhooks/">limit</a> of 50</li> 
</ul> 
<h3>Unified Webhooks</h3> 
<p>Unified webhooks solve this pain point by consolidating all Amplify-related webhooks into a single, comprehensive webhook for your repository. This approach simplifies repository management while ensuring that all associated Amplify apps receive updates and triggers without being constrained by webhook restrictions.</p> 
<h4>Key Benefits</h4> 
<ul> 
 <li>Overcome Git Provider Limitations: Remove current webhook constraints, allowing you to connect as many Amplify apps as needed to a single repository.</li> 
 <li>Enhanced Monorepo Support: Gain more flexibility and efficiency when working with monorepo structures, where multiple projects share a single repository.</li> 
 <li>Simplified Management: Manage multiple Amplify apps with a single repository webhook, reducing complexity and potential points of failure.</li> 
 <li>Improved Workflow Integration: Free up webhook slots for other essential workflows in your development process.</li> 
</ul> 
<h3></h3> 
<h3>Getting Started with Unified Webhooks</h3> 
<p><strong>For New Apps</strong><br /> <a href="https://docs.aws.amazon.com/amplify/latest/userguide/getting-started.html">Deploy a web app</a> with Amplify Hosting. The unified webhook feature will be automatically implemented for your repository.</p> 
<p><strong>For Existing Amplify Users</strong><br /> To take advantage of this new feature, you’ll need to reconnect your repository to your Amplify app. Here’s how:</p> 
<ol> 
 <li>Navigate to your Amplify app in the AWS Management Console.</li> 
 <li>Locate the repository settings for your app.</li> 
 <li>Click on the “Reconnect” button next to your repository information.</li> 
 <li>Confirm the action to replace existing webhooks with the new unified webhook.</li> 
</ol> 
<h3><strong>Walkthrough</strong></h3> 
<p>Here is an example of a current repository with <strong>two Amplify apps</strong> switching to use the new unified webhook.</p> 
<p><img alt="Screenshot of the GitHub webhook settings" class="alignnone size-full wp-image-13868" height="882" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/redactedImage.jpg-scaled.jpeg" width="2560" /></p> 
<p>Each webhook looks like</p> 
<p><img alt="GitHub Webhook payload settings" class="alignnone size-full wp-image-13869" height="1831" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/redactedImage-1.jpg-scaled.jpeg" width="2560" /></p> 
<p>Then go to your Amplify Console, find the <strong>Reconnect Repository</strong> button in <strong>App settings → Branch settings</strong></p> 
<p><img alt="Screenshot of Amplify console and the branch settings area" class="alignnone size-full wp-image-13873" height="1093" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/Amplify-Branch-Settings-scaled.jpeg" width="2560" /></p> 
<p>Click the <strong>Configure GitHub App </strong>and <strong>Complete installation</strong></p> 
<p><img alt="Screenshot of Amplify Console install github app" class="alignnone size-full wp-image-13874" height="698" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/Screenshot-2024-11-20-at-11.20.52 AM.png" width="2868" /></p> 
<p>After a few moments, you will see the repository is switched to the unified webhook.</p> 
<p><img alt="Screenshot of GitHub webhook settings to see the updated webhook from amplify" class="alignnone size-full wp-image-13875" height="1830" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/redactedImage-4.jpg-1-scaled.jpeg" width="2560" /></p> 
<p>You are all set now. Any new amplify app connected to this repository will use the same unified webhook here.</p> 
<h3>Important Considerations</h3> 
<p><strong>Webhook Limit During Migration:</strong> If you’ve already reached the maximum number of webhooks allowed by your Git provider, the automatic migration may not work. In this case, you might need to manually remove at least one existing webhook before reconnecting.</p> 
<p><strong>Regional Operations:</strong> All operations, including the webhook migration, are region-based. This means the migration will only occur for the regional webhooks where you reconnect your Amplify app.</p> 
<h2>Conclusion</h2> 
<p>The introduction of unified webhooks in AWS Amplify Hosting simplifies repository management and enhances support for complex project structures like monorepos. By reducing webhook overhead and streamlining the connection between your git repository and Amplify apps, we’re enabling developers to focus more on building great applications and less on managing infrastructure limitations.</p> 
<p>We’re excited to see how this feature will improve your development workflow, especially for those working with large, complex repositories. Deploy your Next.js, Nuxt, React, Angular, Vue, or other frontend apps in the <a href="https://us-east-1.console.aws.amazon.com/amplify/apps">Amplify Console</a>, and join our community on <a href="http://discord.gg/amplify">Discord</a> to share your thoughts and experiences!</p> 
<p><strong>About the Authors</strong></p> 
<p><img alt="Matt's headshot with a rock and forest behind him" class="wp-image-5165 size-thumbnail alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2021/01/28/matt_A_headshot-150x150.jpg" width="150" /><strong>Matt Auerbach, Senior Product Manager, Amplify Hosting</strong></p> 
<p>Matt Auerbach is a NYC-based Product Manager on the AWS Amplify Team. He educates developers regarding products and offerings, and acts as the primary point of contact for assistance and feedback. Matt is a mild-mannered programmer who enjoys using technology to solve problems and making people’s lives easier. B night, however…well he does pretty much the same thing. You can find Matt on Twitter&nbsp;@mauerbac. He previously worked in Developer Relations at Twitch, Optimizely &amp; Twilio.</p> 
<p><img alt="linson headshot with a national park in the background" class="wp-image-13883 size-thumbnail alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/10/IMG_6173-1-150x150.jpg" width="150" /><strong>Linsong Wang, Software Development Engineer, Amplify Hosting</strong></p> 
<p>Linsong builds features that make it easier for customers to host front-end web applications backed by the reliability and convenience of AWS. In his free time, Linsong enjoys exploring cooking recipes, playing piano, and building life improvement prototype</p>
