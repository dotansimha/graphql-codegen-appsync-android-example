# GraphQL Code Generation for AppSync

This README describes howto test out the repository.

## Make a new Amplify API
Start by populating `schema.graphql`:
```
type Blog @model {
  id: ID!
  name: String!
  posts: [Post] @connection(name: "BlogPosts")
}
type Post @model {
  id: ID!
  title: String!
  blog: Blog @connection(name: "BlogPosts")
  comments: [Comment] @connection(name: "PostComments")
}
type Comment @model {
  id: ID!
  content: String
  post: Post @connection(name: "PostComments")
}
```

Blow away existing ./`amplify` dir, and re-init:
```
rm -r -f ./amplify

amplify init
# choose codegen, codegen, Vim, android, and a valid IAM user

amplidy add api
# GraphQL, codegen, API key, codegen API key, 365, No I am done, yes I have schema
# choose schema.graphl

amplify push
# Say yes and generate all the stuff
```

## Configure Android SDK
Make sure you have `ANDROID_HOME` set, e.g.:
```
export ANDROID_HOME=~/Library/Android/sdk
export JAVA_HOME=/Applications/Android\ Studio.app//Contents/jre/jdk/Contents/Home
```

## Edit `codegen.yml`
You'll need to locate the API URL and API key that you generated
earlier. Put these values into the `codegen.yml`, replacing what's there
right now.

These values were dumped to the console at the end of the `amplify push`
command, earlier. Some additional tips for identifying these value with
the AWS CLI are provided below.

You can see your configured GraphQL APIs quickly with:
```
aws appsync list-graphql-apis
```

And specifically, the API URLs with:
```
aws appsync list-graphql-apis  | jq -r '.graphqlApis[].uris.GRAPHQL'
```

Get the IDs for each API with:
```
aws appsync list-graphql-apis  | jq -r '.graphqlApis[].apiId'
```
Get the API key for one of the APIs:
```
api_id="something from above"
aws appsync list-api-keys --api-id $api_id | jq -r '.apiKeys[].id'
```

## Install Code Generator Dependencies
```
npm install -g yarn
yarn init --yes
yarn global add @graphql-codegen/cli graphql @graphql-codegen/java-apollo-android
```

## Test Run of the Code Gen
Before attempting to build the Android, first ensure that the Code Gen
works.

```
graphql-codegen --config codegen.yml
```

This should output something like:
```
  ✔ Parse configuration
  ✔ Generate outputs
```

Next, run the more ambitious Gradle-integrated version of the above:
```
./gradlew preBuild
```

This should output something like:
```
yarn run v1.19.1
$ graphql-codegen --config codegen.yml
[00:08:15] Parse configuration [started]
[00:08:15] Parse configuration [completed]
[00:08:15] Generate outputs [started]
[00:08:15] Generate to ./app/src/main/java/ (using EXPERIMENTAL preset "java-apollo-android") [started]
[00:08:15] Load GraphQL schemas [started]
[00:08:15] Load GraphQL schemas [completed]
[00:08:15] Load GraphQL documents [started]
[00:08:15] Load GraphQL documents [completed]
[00:08:15] Generate [started]
[00:08:16] Generate [completed]
[00:08:16] Generate to ./app/src/main/java/ (using EXPERIMENTAL preset "java-apollo-android") [completed]
[00:08:16] Generate outputs [completed]
Done in 1.87s.

BUILD SUCCESSFUL in 3s
1 actionable task: 1 executed
```

## Build the Project
```
./gradlew build
```
