# AssignmentTest
Testing internship assignment. Repository will contain several codes, basic programs that will be executed upon PR creation.

### Link to the repository containing source code for the GitHub App:

> https://github.com/Tanishk-Saxena/github-coderunner

### Source Code of the index.js file for probot project:

```

/**
 * This is the main entrypoint to your Probot app
 * @param {import('probot').Probot} app
 */
const axios = require('axios');
const { config } = require("dotenv");
config();

const api_key = process.env.JUDGE0_API_KEY;
const url = "https://judge0-ce.p.rapidapi.com/submissions";
const host = "judge0-ce.p.rapidapi.com";

async function getFiles(owner, repo, pull_number, context) {
  let file = await context.octokit.request('GET /repos/{owner}/{repo}/pulls/{pull_number}/files', {
    owner,
    repo,
    pull_number,
    headers: {
      'X-GitHub-Api-Version': '2022-11-28'
    }
  });
  file = await axios.get(file.data[0].contents_url);
  return file;
}

async function getPR(url) {
  let pr = await axios.get(url);
  pr = pr.data;
  console.log(pr);
  return [pr.base.user.login, pr.base.repo.name, JSON.stringify(pr.number)];
}

async function getOutput(fileExtension, code, context) {
  let token;
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
      let owner = pr.base.user.login;
      let repo = pr.base.repo.name;
      let pull_number = JSON.stringify(pr.number);
      let file = await getFiles(owner, repo, pull_number, context);
      let fileExtension = file.data.name.split('.')[1];
      let code = file.data.content;
      await getOutput(fileExtension, code, context);
    }
  })

  app.on(["issue_comment.created", "issue_comment.edited"], async(context) => {
    let pr = context.payload.issue.pull_request;
    if(pr){
      if(context.payload.comment.body.includes("/execute") && context.payload.comment.user.id !== 139039108){
        let url = pr.url;
        let [owner, repo, pull_number] = await getPR(url);
        let file = await getFiles(owner, repo, pull_number, context);
        let fileExtension = file.data.name.split('.')[1];
        let code = file.data.content;
        await getOutput(fileExtension, code, context);
      }
    }
  })

  // For more information on building apps:
  // https://probot.github.io/docs/

  // To get your app running against GitHub, see:
  // https://probot.github.io/docs/development/
};

```

### To test this app, I will have to host it first, make it available on the github market place, and then any user can install this on their repository and it will monitor the pull requests over that repository. Create a pull request in which you add a file with code written on it. You can test the output in two ways:
1. Either enter the command '/execute' in the PR title itself.
2. Or enter the command '/execute' in the comments under a PR.
Output will be generated each time a PR is opened, or re-opened, or a comment is created, or edited, to contain the command '/execute'.

### Video demo of code execution upon creation of PR

https://github.com/Tanishk-Saxena/AssignmentTest/assets/79462555/7831f811-f49e-4c12-b7a6-0f9643d4a50e

### Summary, Obstacles, Scope for improvement

#### Summary -

This was a really challenging and fun project to make and has certainly opened up a whole new domain for me to explore further. I have learned a lot about the functioning of GitHub, events, Probot, as well as GitHub Apps.

#### Obstacles -

The major obstacle I faced was shortage of time, because I am giving my end sem papers right now so I didn't have enough time to give to this project. So there are still some kinks here and there that could've been approached in a more constructive manner.
Another obstacle I faced was working with PISTON API. It seems to be pretty straightforward, but again I believe due to lack of time, I couldn't give enough time to understand the problems I was facing. The code submitted to the API, must atleast have one UTF file. But for files like C++, I couldn't figure out how to put all the code, increasingly complex codes, into that condensed form. And it didn't accept base64 encoding for just one file, which I felt was rather odd. Nonetheless, I finished the implementation with the help of an API I had built my previous project with, given the confidence I had in it, the familiarity with it, and the time constraint.

#### Future Scope -

There is a big scope for project like this. It could be made to work with multiple files. It can work for many many languages, but for now I have just hard-coded it to work with only C++ files.
