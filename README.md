# AssignmentTest
Testing internship assignment. Repository will contain several codes, basic programs that will be executed upon PR creation.

### Source Code of the index.js file for probot project:

/**
 * This is the main entrypoint to your Probot app
 * @param {import('probot').Probot} app
 */
const axios = require('axios');
const { config } = require("dotenv");
config();

const api_key = process.env.JUDGE0_API_KEY;

module.exports = (app) => {
  // Your code here
  app.log.info("Yay, the app was loaded!");

  app.on("issues.opened", async (context) => {
    console.log("here");
    app.log.info(context);
    const issueComment = context.issue({
      body: "Thanks for opening this issue!",
    });
    return context.octokit.issues.createComment(issueComment);
  });

  app.on(["pull_request.opened", "pull_request.reopened", "pull_request.edited"], async (context) => {
    let pr = context.payload.pull_request;
    let title = pr.title;
    if(title.includes("/execute")){
      let file = await context.octokit.request('GET /repos/{owner}/{repo}/pulls/{pull_number}/files', {
        owner: pr.base.user.login,
        repo: pr.base.repo.name,
        pull_number: JSON.stringify(pr.number),
        headers: {
          'X-GitHub-Api-Version': '2022-11-28'
        }
      });
      // extract code (file data)
      file = await axios.get(file.data[0].contents_url);
      let fileName = file.data.name;
      let fileExtension = fileName.split('.')[1];
      fileName = fileName.split('.')[0];
      let code = file.data.content;
      // fetch output of data
      app.log.info(fileName);
      app.log.info(fileExtension);
      app.log.info(code);
      const url = "https://judge0-ce.p.rapidapi.com/submissions";
      const host = "judge0-ce.p.rapidapi.com";
      let token, output;

      const executeOptions = {
        method: 'POST',
        url: url,
        params: {base64_encoded: 'true', fields: '*'},
        headers: {
          'Content-Type': 'application/json',
          'X-RapidAPI-Key': api_key,
          'X-RapidAPI-Host': host
        },
        data: {
          language_id: 52,  //hard coded for now, for C++
          source_code: code,
        }
      };
      axios.request(executeOptions).then(function async (response) {
          token = response.data.token;
      }).then(()=>{
        setTimeout(()=>{
          const receiveOptions = {
            method: 'GET',
            url: `${url}/${token}`,
            params: {base64_encoded: 'true', fields: '*'},
            headers: {
                'X-RapidAPI-Key': api_key,
                'X-RapidAPI-Host': host
            }
          };
          axios.request(receiveOptions).then(function (response) {
            let buff = new Buffer.from(response.data.stdout, 'base64');
            let stdout = buff.toString('ascii');
            const pullRequestComment = context.issue({
              body: stdout,
            });
            return context.octokit.issues.createComment(pullRequestComment);
          }).catch(function (error) {
              console.error(error);
          });
        }, 5000);
      }).catch(function (error) {
          console.error(error);
      });
    }
  })

  app.on(["issue_comment.created", "issue_comment.edited"], async(context) => {
    console.log("here");
    app.log.info(context);
    let comment = context.payload.comment;
    let issue = context.payload.issue;
    app.log.info(issue);
    app.log.info(comment);
    if(comment.includes("/execute")){
      console.log("execute found in comment");
      // run this code
    }
  })

  // For more information on building apps:
  // https://probot.github.io/docs/

  // To get your app running against GitHub, see:
  // https://probot.github.io/docs/development/
};

### To test this app, I will have to host it first, make it available on the github market place, and then any user can install this on their repository and it will monitor the pull requests over that repository.

### Video demo of code execution upon creation of PR

https://github.com/Tanishk-Saxena/AssignmentTest/assets/79462555/7831f811-f49e-4c12-b7a6-0f9643d4a50e

### Summary, Obstacles, Scope for improvement

#### Summary -

This was a really challenging and fun project to make and has certainly opened up a whole new domain for me to explore further. I have learned a lot about the functioning of GitHub, events, Probot, as well as GitHub Apps.

#### Obstacles -

The major obstacle I faced was shortage of time, because I am giving my end sem papers right now so I didn't have enough time to give to this project. It is because of this reason only, that I could not completely implement the required functionality before deadline. I couldn't get it to work with comments, whenever we write the command /execute in comments, no event is caught and consequently no execution takes place. Unfortunately I fell short of time, otherwise with just half a day more, I could've probably finished this project completely.
Another obstacle I faced was working with PISTON API. It seems to be pretty straightforward, but again I believe due to lack of time, I couldn't give enough time to understand the problems I was facing. The code submitted to the API, must atleast have one UTF file. But for files like C++, I couldn't figure out how to put all the code, increasingly complex codes, into that condensed form. And it didn't accept base64 encoding for just one file, which I felt was rather odd. Nonetheless, I finished the implementation with the help of an API I had built my previous project with, given the confidence I had in it, the familiarity with it, and the time constraint.

#### Future Scope -

There is a big scope for project like this. It could be made to work with multiple files. I still have to implement the functionality of working with comments commands. It can work for many many languages, but for now I have just hard-coded it to work with only C++ files.
