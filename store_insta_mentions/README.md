### 1. How to auto-collect Instagram Story mentions and log them in Notion for influencer tracking

- **Category:** Social Media
- **Rating:** 77/100
- **Difficulty:** Easy-Beginner
- **Target Audience:** Social media managers

## 📖 Complete Tutorial Content

### Tutorial 1: How to Auto-Collect Instagram Story Mentions and Log Them in Notion

**Hook:**
> Struggling to keep track of all your brand’s Instagram Story mentions? Manually screenshotting and logging each one is time-consuming and error-prone. This tutorial shows you how to automate the entire process using n8n and the Instagram Graph API.

**Step-by-Step:**
**Step 1: Set Up Your Meta Developer App**
To access Instagram Story mentions, you need to use the Instagram Graph API via Meta’s developer tools.
  1. Go to https://developers.facebook.com/ and create a new App (type: Business).
  2. Add the Instagram Graph API product to your app.
  3. In App Roles, add yourself as an Administrator.
  4. Generate a User Access Token with the ‘instagram_graph_user_media’ and ‘pages_show_list’ permissions.
  5. Make sure your Instagram account is connected to a Facebook Page under Business Settings.
💡 Tip: Use the Access Token Debugger on Facebook to extend your token to 60 days for longer-lasting automations.

**Step 2: Create a Notion Database to Log Mentions**
Set up a Notion table that will store each mention, including the username, timestamp, and story URL.
  1. In Notion, create a new database (Table view) named ‘Influencer Mentions’.
  2. Add columns: ‘Username’ (text), ‘Timestamp’ (date), ‘Story URL’ (URL), and optionally ‘Notes’.
  3. Copy the database’s internal ID from the Notion Share menu (you’ll need this for the API).
💡 Tip: Make sure to share the database with your Notion integration so n8n can access it.

**Step 3: Get Your Notion Integration Token**
You’ll need a Notion integration token to allow n8n to write into your database.
  1. Go to https://www.notion.com/my-integrations and create a new integration.
  2. Give it read/write access and copy the generated token.
  3. Share your ‘Influencer Mentions’ database with this integration.
💡 Tip: Use a secure password manager to store your Notion token safely.

**Step 4: Set Up a Scheduled Trigger in n8n**
Instagram’s API doesn’t push mentions, so we’ll poll for them every 15 minutes using a Schedule Trigger.
  1. In n8n, create a new workflow and drag in the ‘Schedule Trigger’ node.
  2. Set it to run every 15 minutes or as needed for your use case.
  3. Name your workflow something like ‘Instagram Mentions to Notion’.
💡 Tip: You can adjust the frequency based on how often you get tagged—more frequent polling is more responsive but uses more API calls.

**Step 5: Use HTTP Request to Fetch Story Mentions from Instagram**
Since n8n doesn’t have a built-in Instagram node, we’ll use the HTTP Request node to call the Instagram Graph API.
  1. Add an ‘HTTP Request’ node after the Schedule Trigger.
  2. Set it to GET and use this URL: `https://graph.instagram.com/me?fields=mentioned_media{media_url,timestamp,username}&access_token=YOUR_TOKEN`
  3. Replace `YOUR_TOKEN` with your long-lived access token.
  4. Set the Response Format to ‘JSON’.
💡 Tip: Use the ‘Execute Node’ button to test this and inspect the output structure—this will help with mapping fields later.

**Step 6: Extract Mention Details Using a Set Node**
Before sending data to Notion, we’ll clean it up by selecting only the relevant fields.
  1. Add a ‘Set’ node after the HTTP Request.
  2. Map the output fields: ‘username’, ‘timestamp’, and ‘media_url’.
  3. Rename ‘media_url’ to ‘Story URL’ for clarity.
💡 Tip: If multiple mentions are returned, use a SplitInBatches node to process them one at a time.

**Step 7: Send the Mention to Notion**
We’ll now push the cleaned data into your Notion database using n8n’s Notion node.
  1. Add a ‘Notion’ node set to ‘Page Create’.
  2. Select your Notion credentials or create new ones using your integration token.
  3. Choose your ‘Influencer Mentions’ database.
  4. Map the fields: Username → Username, Timestamp → Timestamp, Story URL → Story URL.
💡 Tip: Be sure the Notion database column types match (e.g., URL for Story URL) to avoid validation errors.

**Step 8: Test Your Workflow and Activate It**
Now it’s time to test the full workflow and set it live.
  1. Click ‘Execute Workflow’ to run it manually and confirm a mention is logged in Notion.
  2. If everything looks good, toggle the ‘Active’ switch in the top right.
  3. Your workflow will now run automatically on the schedule you defined.
💡 Tip: Use the n8n Execution Logs to monitor for any failed runs or API changes.

**Required Tools:**
• n8n: To build and automate the workflow
• Meta for Developers: To access Instagram Graph API
• Notion: To store and organize mentions

**Common Pitfalls:**
⚠️ Instagram account not connected to a Facebook Page
   ✅ Go to Business Settings in Meta and ensure the Instagram account is linked to a Facebook Page.
⚠️ Access token expires after an hour
   ✅ Use Facebook’s Access Token Debugger to generate a long-lived token (60 days).
⚠️ Notion node fails to create page
   ✅ Check that your Notion database column types match the data being sent (e.g., URL for links).

**Troubleshooting:**
Issue: Workflow runs but no mentions appear in Notion
Fix: Double-check the Graph API endpoint and test it using a manual request in a browser or Postman.

Issue: 403 Forbidden error from Instagram API
Fix: Ensure your token has the ‘instagram_graph_user_media’ permission and the account is properly linked.

**Checklist:**
☐ Instagram account is set up as Business or Creator and linked to Facebook
☐ Meta App is created and has the correct permissions
☐ Notion integration and database are correctly configured and shared
☐ n8n workflow is tested and active
☐ Access tokens are valid and stored securely

**Developer Mention**
if this helps you save time or money please consider helping fund future projects contributions are appreciated
visit https://kofi.com/colin8080 to buy me a coffee thanks..
