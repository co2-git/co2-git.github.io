# Importing a MySQL file using NodeJS

In this post we'll see how to do in NodeJS the following bash code:

    mysql -h $mysql_host -u $mysql_user -p$mysql_password $mysql_database < dump.sql
    
*For the sake of example, we'll say that your MySQL credentials are stored as ENVIRONMENT variables.*
    
    module.exports = function importSQLFile (sqlFile, callback) {

      // opening the SQL File for reading
      var stdin = require('fs').createReadStream(sqlFile, { encoding: 'utf-8' });

      // Calling mysql
      var spawn = require('child_process').spawn('mysql', [
        '-h', process.env.mysql_host,
        '-u', process.env.mysql_user,
        '-p' + process.env.mysql_password,
        process.env.mysql_database]);

      // Catching error
      spawn.on('error', callback);

      // Catching unexpected exit
      spawn.on('exit', function (signal) {
        if ( typeof signal === 'number' && signal ) {
          return callback(new Error('Importing SQL file failed with code ' + signal));
        }

        if ( typeof signal === 'string' ) {
          return callback(new Error('Importing SQL file failed with signal ' + signal));
        }
      });

      // Piping the SQL file contents to MySQL
      stdin.on('data', function (data) {
        spawn.stdin.write(data);
      });

      // Returning on stream end
      stdin.on('end', function () {
        callback();
      })
    }
