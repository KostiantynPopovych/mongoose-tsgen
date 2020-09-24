mongoose-tsgen
==============

A Typescript interface generator for Mongoose that works out of the box.

[![oclif](https://img.shields.io/badge/cli-oclif-brightgreen.svg)](https://oclif.io)
[![Version](https://img.shields.io/npm/v/mongoose-tsgen.svg)](https://npmjs.org/package/mongoose-tsgen)
[![Downloads/week](https://img.shields.io/npm/dw/mongoose-tsgen.svg)](https://npmjs.org/package/mongoose-tsgen)
[![License](https://img.shields.io/npm/l/mongoose-tsgen.svg)](https://github.com/francescov1/mongoose-tsgen/blob/master/package.json)

Requires only a few lines of additional code to support, and is compatible with all Mongoose features. This CLI works by importing your schema definitions, parsing the structure and generating an index.d.ts file containing interfaces for all your schemas. A section for custom interfaces and types is provided at the bottom of `index.d.ts`. This will remain untouched when re-generating the interfaces unless the `--fresh` flag is provided.

### Example

NOTE: The CLI supports Mongoose Models written in both Typescript and Javascript. Examples are written in Typescript.

#### user.ts

```typescript
// NOTE: you will need to import these types after your first ever run of the CLI
// See the 'Initializing Schemas' section
import mongoose, { IUser, IUserModel } from "mongoose";
const { Schema } = mongoose;

const UserSchema = new Schema({
  email: {
    type: String,
    required: true
  },
  firstName: {
    type: String,
    required: true
  },
  lastName: {
    type: String,
    required: true
  },
  metadata: Schema.Types.Mixed,
  friends: [
    {
      uid: {
        type: Schema.Types.ObjectId,
        ref: "User",
        required: true
      },
      nickname: String
    }
  ],
  city: {
    coordinates: {
      type: [Number],
      index: "2dsphere"
    }
  }
});

// NOTE: `this: IUser` is for the "fake this" TS feature
// you will need to add these in after your first ever run of the CLI to tell TS the type of 'this' value in scope of the getter
UserSchema.virtual("name").get(function(this: IUser) { return `${this.firstName} ${this.lastName}` });

// method functions
UserSchema.methods = {

  // NOTE: `this: IUser` is for the "fake this" TS feature
  // you will need to add these in after your first ever run of the CLI to tell TS the type of 'this' value in scope of the method function
  isMetadataString(this: IUser) {
    return typeof this.metadata === "string";
  }

}

// static functions
UserSchema.statics = {

  // NOTE: `this: IUserModel` is for the "fake this" TS feature.
  // you will need to add these in after your first ever run of the CLI to tell TS the type of 'this' value in scope of the static function
  async getFriends(
    this: IUserModel,
    // An array of user _id properties. You could also use `friendUids: ObjectId[]` here
    friendUids: IUser["_id"][]
  ): Promise<any[]> {
    const friends = await this.aggregate([
      { $match: { _id: { $in: friendUids } } }
    ]);

    return friends;
  }

}

// NOTE: you will need to provide the IUser and IUserModel types when initializing the Mongoose model. 
// The return parameter is of type IUserModel.
// See the 'Initializing Schemas' section
export const User: IUserModel = mongoose.model<IUser, IUserModel>("User", UserSchema);
export default User;
```

#### generated index.d.ts

```typescript
// ######################################## THIS FILE WAS GENERATED BY MONGOOSE-TSGEN ######################################## //

// NOTE: ANY CHANGES MADE WILL BE OVERWRITTEN ON SUBSEQUENT EXECUTIONS OF MONGOOSE-TSGEN.
// TO ADD CUSTOM INTERFACES, DEFINE THEM IN THE `CUSTOM INTERFACES` BLOCK

import mongoose from "mongoose";
type ObjectId = mongoose.Types.ObjectId;

declare module "mongoose" {

	interface IUserFriend extends mongoose.Types.Subdocument {
		uid: IUser["_id"] | IUser;
		nickname?: string;
	}

	export interface IUserModel extends Model<IUser> {
		getFriends: Function;
	}

	export interface IUser extends Document {
		email: string;
		metadata?: any;
		firstName: string;
		lastName: string;
		friends: Types.DocumentArray<IUserFriend>;
		cityCoordinates?: Types.Array<number>;
		name: any;
		isMetadataString: Function;
	}

// ########################################## CUSTOM INTERFACES ########################################## //
// ######################################## END CUSTOM INTERFACES ######################################## //
}
```

### Initializing Schemas

Once you've generated your index.d.ts file, all you need to do is add the following types to your schema definitions:

* Before:

```typescript
import mongoose from "mongoose";

const UserSchema = new Schema(...);

export const User = mongoose.model("User", UserSchema);
export default User;
```

* After:

```typescript
import mongoose, { IUser, IUserModel } from "mongoose";

const UserSchema = new Schema(...);

export const User: IUserModel = mongoose.model<IUser, IUserModel>("User", UserSchema);
export default User;
```

Then you can import the interfaces across your application from the Mongoose module and use them for document types:

```typescript
// import interface from mongoose module
import { IUser } from "mongoose"

async function getUser(uid: string): IUser {
  // user will be of type IUser
  const user = await User.findById(uid);
  return user;
}

async function editEmail(user: IUser, newEmail: string): IUser {
  user.email = newEmail;
  return await user.save();
}
```

<!-- toc -->
* [Installation](#installation)
* [Commands](#commands)
<!-- tocstop -->
# Installation
<!-- usage -->
```sh-session
$ npm install -g mongoose-tsgen
$ mtgen COMMAND
running command...
$ mtgen (-v|--version|version)
mongoose-tsgen/0.0.1 darwin-x64 node-v14.3.0
$ mtgen --help [COMMAND]
USAGE
  $ mtgen COMMAND
...
```
<!-- usagestop -->
# Commands
<!-- commands -->
* [`mtgen help [COMMAND]`](#mtgen-help-command)
* [`mtgen run [PATH]`](#mtgen-run-path)

## `mtgen help [COMMAND]`

display help for mtgen

```
USAGE
  $ mtgen help [COMMAND]

ARGUMENTS
  COMMAND  command to show help for

OPTIONS
  --all  see all commands in CLI
```

_See code: [@oclif/plugin-help](https://github.com/oclif/plugin-help/blob/v3.2.0/src/commands/help.ts)_

## `mtgen run [PATH]`

Generate an index.d.ts file containing Mongoose Schema interfaces. If no `path` argument is provided, the tool will expect all models to be exported from `./src/models` by default.

```
USAGE
  $ mtgen run [PATH]

OPTIONS
  -d, --dry-run        Print output rather than writing to file
  -f, --fresh          Fresh run, ignoring previously generated custom interfaces
  -h, --help           show CLI help
  -o, --output=output  [default: ./src/types/mongoose] Path of output index.d.ts file
```

_See code: [src/commands/run.ts](https://github.com/Bounced-Inc/mongoose-tsgen/blob/v0.0.1/src/commands/run.ts)_
<!-- commandsstop -->

### Coming Soon (most of the following features are already supported but use looser typing than likely desired):

- Methods and statics parameter types. Currently these are typed as `Function`.
- Support for `Model.Create`. Currently `new Model` must be used.
- Support for setting subdocument properties without casting to any. When setting a subdocument array, Typescript will yell at you if you try and set them directly (ie `user.friends = [{ uid, name }]`) as it expects the array to contain additional subdocument properties. For now, this can be achieved by writing `user.friends = [{ uid, name }] as any`.
- A Mongoose plugin. This will remove the need to re-run the CLI when changes to your schema are made.

Would love any help with the listed features above.
