# Offline Conversion Automated Uploader
Offline Conversion Automated Uploader is a command line tool that helps Facebook advertisers and marketing partners upload offline transactions to the [Facebook marketing API](https://developers.facebook.com/docs/marketing-api/offline-conversions) without building their own application for API Integration.

## Why use this tool?
* Building API integration will require engineering resources. Typically, an engineer without experience working with our Facebook marketing API will need about 3 weeks for development and testing.
* In order to achieve the best possible match between your customers and Facebook users, the data needs to be normalized and hashed correctly. MDFU tool uses the libraries written by Facebook to ensure the best possible match rate.
* For any issues with this tool, you will get support from Facebook.
* This tool will be updated periodically to support more features.

## Prerequisites
* An offline event set. [How to create](https://www.facebook.com/business/help/339320669734609)
* Your offline event file in CSV format. [Examples](https://github.com/facebookincubator/offline-conversion-file-uploader/blob/master/demo/test-event-files.zip)
* **Strongly recommended:** manually upload your offline events via [UI](https://business.facebook.com/events_manager/?selected_data_sources=DATA_SET) to familiarize yourself with this product.
* System user access token. [Follow step 2 and 3 of this guide.](https://developers.facebook.com/docs/marketing-api/offline-conversions). Not required if you only use this tool to hash the data.

## How to use

### Install

Make sure git and node are installed and are up-to-date on your machine, then run:

```bash
git clone https://github.com/facebookincubator/offline-conversion-file-uploader
npm install
npm run compile
```

###Examples

Before uploading real data, you can create a test data set and upload some test data to familiarize yourself with this tool. Take a look at this [guide](https://github.com/facebookincubator/offline-conversion-file-uploader/blob/master/demo/demo.md) and have a try.

### Run Command

Use `validate` command when you are setting up the tool and config. Check the report generated by this command and resolve issues. Once you are done with configuration, use `upload` command to upload events.

If you want to separate the hashing and uploading into two steps. You can use `preprocess` command first to hash the file, and then use `upload-preprocessed` command to upload the hashed file.

```
node lib/cli.js <COMMAND> --configFilePath <PATH_TO_CONFIG> <...OTHER_PARAMS>
```

See the **Commands** section for details of each command. See the previous section for some sample commands.

### Schedule

Once you make sure an upload is made successfully, you can automate your upload by scheduling the command. We recommend to use **crontab** for POSIX systems, and use **Powershell** and **Task Scheduler** for Windows.

## Commands
|Command|Description|Options (Required options are in bold)|
|-------|-----------|--------------------------------------|
|upload|Hash and upload the CSV file to Facebook.|**`accessToken`** **`configFilePath`** **`dataSetID`** **`inputFilePath`** **`mapping`** `batchSize` `customTypeInfo`  `delimiter` `format` `header` `ignoreSampleErrors` `logging` `namespaceID` `presetValues` `reportOutputPath` `skipRowsAlreadyUploaded` `uploadID` `uploadTag` `uploadTagPrefix`|
|validate|Do a dry-run and validate some sample rows.|**`accessToken`** **`configFilePath`** **`dataSetID`** **`inputFilePath`** **`mapping`** `customTypeInfo` `delimiter` `format` `header` `logging` `namespaceID` `numRowsToValidate` `presetValues` `reportOutputPath`|
|preprocess|Hash the CSV file and store it locally without sending it Facebook. It also normalizes other fields, such as converting ISO-formatted event time into unix time.|**`inputFilePath`** **`configFilePath`** **`mapping`** `customTypeInfo` `delimiter` `format` `header` `ignoreSampleErrors` `logging` `preprocessOutputPath` `presetValues` `reportOutputPath`|
|upload-preprocessed|Upload the hashed CSV generated by `preprocess` command.|**`accessToken`** **`dataSetID`** **`inputFilePath`** `batchSize` `configFilePath` `ignoreSampleErrors` `logging` `namespaceID` `reportOutputPath` `skipRowsAlreadyUploaded` `uploadID` `uploadTag` `uploadTagPrefix`|

## Options
###How to Specify

Most options can be specified in both ways.

For example, if you want to specify `accessToken`, you can either do it by passing it directly to CLI command

``` bash
node lib/cli.js <COMMAND> --accessToken <YOUR_ACCESS_TOKEN> ...
```

or by setting it in the config JSON specified by `configFilePath`.

```json
{
    "accessToken": <YOUR_ACCESS_TOKEN>,
    ...
}
```

For boolean options like `skipRowsAlreadyUploaded`, when specifying in CLI, there is no need to explicitly set it to `true`

```bash
node lib/cli.js <COMMAND> --skipRowsAlreadyUploaded ...
```

###All Options

Here is a list of all options we support. See previous section for options supported/required by each command. See **Resuming Support** to see why `uploadTag`/`uploadTagPrefix`/`uploadID` are deprecated.

| Option      | Description | Where to specify | Default | Example |
| ----------- | ----------- | --------------- | ------- | ----------- |
| accessToken | Access token required to make API calls to Facebook server. | CLI or Config JSON | *No default* | `"EAAC...T4ZD"` |
| dataSetID | ID of offline event set you want to upload events to. | CLI or Config JSON | *No default* | `"123456789"` |
| uploadTag | **(Deprecated)**  Tag to identify the events uploaded. Should use unique string for each distinct file uploaded. | CLI or Config JSON | *No default* | `"Offline Conversions"` |
| uploadTagPrefix | **(Deprecated)**  Instead of providing uploadTag, you can also define prefix (ex: Offline Conversions), then the tool will append filename/timestamp and use it as the uploadTag. If uploadTag is set, uploadTagPrefix is ignored. Example: *Offline Conversions (example_events_big_100k.csv@1493837377000)* | CLI or Config JSON | *No default* | `"Offline Conversions"` |
| uploadID | **(Deprecated)** ID of the uploadTag. | CLI or Config JSON | *No default* | `"123456789"` |
| skipRowsAlreadyUploaded | Rows will be skipped if part of the same file was already uploaded before. | CLI or Config JSON | `false` | `true` |
| ignoreSampleErrors | The command will be stopped if there are too many errors in first 1,000 rows when this options is not specified. Use this options to override the behavior and to forcefully continue execution. | CLI or Config JSON | `false` | `true` |
| inputFilePath | Path of input CSV. | CLI or Config JSON | *No default* | `"path/to/offline_data.csv"` |
| batchSize | Configures how many events are sent to Facebook server in one API call. Ranges from 1 to 2000. Lower the number if network is slow or unstable. | CLI or Config JSON | `2000` | `500` |
| delimiter | Delimiter of CSV file. | CLI or Config JSON | `","` | `"\t"` |
| header | Whether to mark the first row as header and skip it when uploading. | CLI or Config JSON | `false` | `true` |
| mapping | Defines what each column in the file is for. | Config JSON Only | *No default* |See the **mapping** section below.|
| format | Provide more information about the formatting of certain types of columns. | Config JSON Only | `{}` | See the **mapping** section below. |
| customTypeInfo | Provide more information for each custom data column in the mapping. | Config JSON Only | `{}` | See the **mapping** section below. |
| presetValues | Provide default value for missing mapping. | Config JSON Only | `{}` | See the **mapping** section below. |
| logging | Level of logging. See supported levels [here](https://www.npmjs.com/package/winston#logging-levels). | CLI or Config JSON | `"verbose"` | `"debug"` |
| namespaceID | Namespace ID for third-party ID mapping. | CLI or Config JSON | No default | `"123456789"` |
| numRowsToValidate | Max number of sample rows to validate when running `validate` command. Ranges from 1 to 1000. | CLI or Config JSON | `1000` | `500` |
| preprocessOutputPath | Path of output of `preprocess` command. Hashed events will be generated into the path. The file will be truncated if it already exists. | CLI or Config JSON | `"preprocess-output.csv"` | `"hashed.csv"` |
| reportOutputPath | Path of output report of each command. The report contains summary, issues and error samples. The file will be truncated if it already exists. | CLI or Config JSON | `"report.txt"` | `"upload-report.txt"` |

### Mapping

The `mapping` field is a dictionary that maps the index of a column to offline event schema. Here is an example:

```json
"mapping": {
  "0": "match_keys.email",
  "1": "match_keys.phone",
  "2": "event_time",
  "3": "event_name",
  "4": "value",
  "5": "currency",
  "7": "custom_data.sales_type",
  "9": "custom_data.department"
}
```

The key of `mapping` is the index of column, starting with 0. The value of mapping field can be one of the values listed in the following table:

| Mapping           | Is Required                                                  | Description                                                  |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| match_keys.XXXXX  | Required.                                                    | The identifier info used to match people. XXXXX needs to be replaced with the match key type such as email, phone, etc. For list of available match key types, please see 'Key name' column in [this table](https://developers.facebook.com/docs/marketing-api/offline-conversions#match-keys). |
| event_time        | Required.                                                    | Time when the offline conversion happened. The data in the corresponding column is recommend to be formatted in ISO8601 (example: `2018-03-09T19:17:19.345Z`) or unix time (example: `1520623023`). |
| event_name        | Required. Can be either specified in `mapping` or in `presetValues`. | See `event_name` row in the [data parameters table](https://developers.facebook.com/docs/marketing-api/offline-conversions/#data-parameters). |
| value             | Required if exists event having `Purchase` as `event_name`. Optional for other cases. | Value of conversion event. Required for Purchase event. We do not accept 0 and negative values. Example: `16.00`. |
| currency          | Required if `value` is mapped. Can be either specified in `mapping` of in `presetValues`. | Three-letter ISO currency for this conversion event. Required when and only when value presents. |
| order_id          | Optional but recommended. Required if `item_number` is mapped. | The unique ID associated with each of your transactions.     |
| item_number       | Optional.                                                    | Item number is to distinguish different items within same order. |
| custom_data.XXXXX | Optional.                                                    | Additional information about the conversion event. For example, send store location ID as custom_data.location_id. |

We also provide a `presetValues` option for your convenience if your CSV file does not contain a column for `event_name` or `currency`. For example, if all of your events are purchases and uses USD as currency, you can use the following presetValues:

```json
"presetValues": {
    "event_name": "Purchase",
    "currency": "USD"
}
```

Note that you cannot specify a field's value in both `presetValues` and `mapping`. For example, in the use case above, you are not allowed to map any column to `event_name` or `currency` in your `mapping`, since they are already specified in `presetValues`.

For some specific mapping, namely `match_keys.dob` and `event_time`, you also need to specify the format of your data in the `format` option.

For `match_keys.dob` we support the following format:

- `MM/DD/YYYY` `MMDDYYYY` `MM-DD-YYYY`
- `DD/MM/YYYY` `DDMMYYYY` `DD-MM-YYYY`
- `YYYY/MM/DD` `YYYYMMDD` `YYYY-MM-DD`
- `MM/DD/YY` `MMDDYY` `MM-DD-YY`
- `DD/MM/YY` `DDMMYY` `DD-MM-YY`
- `YY/MM/DD` `YYMMDD` `YY-MM-DD`

And for `event_time` we support `ISO8601` and `unix_time`. We also support formats listed above for `event_time`, but that's not recommended, as we recommend `event_time` accurate to minutes or seconds.

For `event_time`, we provide an additional option to offset the time zone. For `ISO8601` and `unix_time` time zone is not needed, so please set it to 0. For other formats, you can set it to an number that represents offset (in hours) that should be applied to the `event_time`.

Here is an example of `format`, note that the format of `event_time` and `match_keys.dob` differs slightly:

```json
"format": {
  "event_time": {
    "timeFormat": "ISO8601"
  },
  "dob": "MM/DD/YYYY"
}
```

The last piece of mapping is `customTypeInfo`. If you don't map any custom data fields, please leave it unspecified or as an empty dictionary `{}`. If you mapped custom data field, for each custom data field, you need to specify its type to be either `string` or `number` type. For example, if you've mapped `custom_data.margin_value` and `custom_data.department_name`, you need to set `customTypeInfo` to

```json
"customTypeInfo": {
  "margin_value": {
    "baseType": "number"
  },
  "department_name": {
    "baseType": "string"
  }
}
```

###Resuming Support

Resuming is a way to ensure each row is uploaded once and only once, regardless of process or machine crashes, or network failures. The following prerequisites should be satisfied to support resuming:

- You should use different file names for each input file. A way to guarantee that is to add a date stamp at each file. For example, you use `Offline conversions 2018-03-01.csv` to represent sales happen that day (or that week).
- Once the input file is generated, you don't change it.
- The anti-pattern is to use one file name for all events, and you update the file for new events. **Don't do this.**

Then when you call the script to upload the events, make sure:

- Do **not** specify `uploadTag`/`uploadTagPrefix`/`uploadID`. We'll automatically generate an `uploadTag` for you based on the name and size of input file, we use the auto generated `uploadTag` to discover the range of events that were already uploaded.
- Set `skipRowsAlreadyUploaded`. This options will allow the uploader to skip the rows that falls into the ranges uploaded before.

## FAQ:

###Q: There's another tool called MDFU, which one should I use?

We previously built a tool named [MDFU](https://github.com/facebookincubator/marketing-data-file-uploader) with similar functionality.
MDFU can be used to upload offline conversions and custom audiences. Our new tool focuses on uploading offline conversions only, but provides additional functionality.

* It generates a report to help troubleshooting issues with your data file.
* It is more robust, because it uses the battle-proven uploader core that is used in the web version.
* It supports separating upload and preprocessing into two steps, so you can verify that data is hashed properly before sending to Facebook.
* It supports a validate command which does a dry-run on some sample rows of your data file, so you have a chance to fix the issue before sending them to Facebook.
* It supports resuming, so you can upload same file again without causing duplications.

In short, if you want to upload offline conversions, use this tool. If you want to upload custom audiences, use MDFU.

### Q: My company has firewall which will block API calls to Facebook. What are my options?

- Whitelist Facebook IP's: Contact your security team to whitelist IP addresses returned by this command:

  ```
  whois -h whois.radb.net -- '-i origin AS32934' | grep ^route
  ```

  For more information, please refer to [this guide](https://developers.facebook.com/docs/sharing/webmasters/crawler) which explains whitelisting for Facebook crawlers, but the same set of IP's are used for API servers.

- Request your security team to create DMZ where outbound HTTP request is allowed.

###Q: How this tool works?

This is a node.js application that will go through following steps to upload your offline conversions to Facebook's marketing API. Here is how the tool uploads data:

1. Read configurations.
2. Read input file in stream.
3. For each line read, columns are normalized and hashed for upload.
4. Collect normalized and hashed data into batches.
5. POST each batch to the API endpoint.

## License
Facebook Offline Conversion Automated Uploader is BSD-licensed. We also provide an additional patent grant.
