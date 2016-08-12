# Summary

You can configure a parallel export of view data to an FTP or an SFTP server using `exportTo()` clause with `Ftp` in a view. Whenever the view performs its transformation successfully and materializes, Schedoscope triggers a mapreduce job that 
* splits up the view's data in chunks;
* in parallel encodes the data in those chunks as JSON or CSV;
* aggregates and optionally compresses the chunks into separate files;
* and finally copies the chunk files in parallel via (S)FTP to the specified server.

# Syntax

    def Ftp(
      v: View,
      ftpEndpoint: String,
      ftpUser: String,
      ftpPass: String = null,
      filePrefix: String = null,
      delimiter: String = "\t",
      printHeader: Boolean = true,
      keyFile: String = "~/.ssh/id_rsa",
      fileType: FileOutputType = FileOutputType.csv,
      numReducers: Int = Schedoscope.settings.ftpExportNumReducers,
      passiveMode: Boolean = true,
      userIsRoot: Boolean = true,
      cleanHdfsDir: Boolean = true,
      exportSalt: String = Schedoscope.settings.exportSalt,
      codec: FileCompressionCodec = FileCompressionCodec.gzip,
      isKerberized: Boolean = !Schedoscope.settings.kerberosPrincipal.isEmpty(),
      kerberosPrincipal: String = Schedoscope.settings.kerberosPrincipal,
      metastoreUri: String = Schedoscope.settings.metastoreUri)

# Description

Here is a description the parameters you must or can pass to `Ftp` exports:

- `v`: a reference to the view being exported, usually `this` (mandatory)
- `ftpEndpoint`: an `ftp://` or `sftp://` URL pointing to the (S)FTP server to which the view's data are to be copied (mandatory)
- `ftpUser`: the (S)FTP user for connecting to the server (mandatory) 
- `ftpPass`: the (S)FTP user's password or SSH key passphrase. No password by default
- `filePrefix`: the filename prefix of the resulting files being copied. The default is the Hive database concatenated with the Hive table name using `-`
- `delimiter`: in case the view's data is to be exported in CSV format, this specifies the CSV delimiter to use. The default is `\t`
- `keyFile`: the location of the private SSH key file when SFTP transport with key-based authentication is used. Defaults to `~/.ssh/id_rsa`
- `fileType`: specifies the export format, either `FileOutputType.csv` or `FileOutputType.json`
- `numReducers`: the number of reducers to use during the exports. This defines the parallelism of the export, i.e., how many chunk files to generate for the view export. Defaults to the config setting `schedoscope.export.ftp.numberOfReducers` (i.e., 10)
- `passiveMode`: specifies whether to use FTP passive mode or not, defaults to `true`
- `userIsRoot`: specifies whether the FTP root directory after login is the user directory (`true`) or `/`(`false`). Default is `true`
- `cleanHdfsDir`: clean up the temporary HDFS dir used during export. Default is `true`
- `exportSalt`: optional salt to use when anonymizing fields during export. Defaults to `schedoscope.export.salt` in the configuration 
- `codec`: the compression codec to use: either `FileCompressionCodec.none`, `FileCompressionCodec.gzip` or `FileCompressionCodec.bzip2`, defaults to `FileCompressionCodec.gzip`
- `isKerberized`: is this a kerberized cluster environment? Defaults to `true`, if `schedoscope.kerberos.principal` is set in the configuration
- `kerberosPrincipal`: the Kerberos principal to use in a Kerberos cluster enviroment. Defaults to `schedoscope.kerberos.principal` as set in the configuration
- `metastoreUri`: the connection URI to use for the Hive metastore. Defaults to `schedoscope.metastore.metastoreUri` as set in the configuration

 
# Example
    
    package test.export

    case class ClickWithJdbcExport(
      year: Parameter[String],
      month: Parameter[String],
      day: Parameter[String]) extends View
           with DailyParameterization {

      val id = fieldOf[String]
      val url = fieldOf[String]

      val stage= dependsOn(() => Stage(year, month, day))

      transformVia(
        () => HiveTransformation(
          insertInto(this, s"""SELECT id, url FROM ${stage().tableName} WHERE date_id='${dateId}'""")))

       // This will create a table named dev_test_export_click_with_jdbc_export in the MySQL database test_db
       // (assuming an environment named dev).
       // The columns of this table would be:
       // id text, url text, year text, month text, day text, date_id text, used_filter text
       exportTo(() => Jdbc(this, "jdbc:mysql://localhost:3306/test_db", "root", "root"))

    }
