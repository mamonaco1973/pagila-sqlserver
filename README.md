# Steps to Deploy Pagila SQL Server Database

```bash
#!/bin/bash
# ===================================================================================================
# PURPOSE:
#   This script installs Microsoft SQL Server CLI tools (sqlcmd),
#   executes multiple SQL scripts against a target SQL Server instance.
# ===================================================================================================

# ===========================
# INSTALL SQL SERVER CLI TOOLS
# ===========================

# Download Microsoft's public signing key and save it in the system's trusted keyring directory.
# This ensures that packages downloaded from Microsoft repositories can be verified for authenticity.
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc > /dev/null

# Add Microsoft's Ubuntu package repository for SQL Server tools to the system's sources list.
# This makes 'mssql-tools' available for installation via apt.
echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/22.04/prod jammy main" | sudo tee /etc/apt/sources.list.d/mssql-release.list

# Update the list of available packages to include the newly added Microsoft repository.
sudo apt-get update -y

# Install 'mssql-tools' (contains sqlcmd, bcp) and 'unixodbc-dev' (ODBC driver dependencies).
# 'ACCEPT_EULA=Y' automatically accepts Microsoft's license terms to avoid interactive prompts.
sudo ACCEPT_EULA=Y apt-get install -y mssql-tools unixodbc-dev > /tmp/mssql-tools.log

# Extend PATH to include the directory where sqlcmd and bcp are installed.
# This allows these commands to be run from any location in the shell session.
export PATH="$PATH:/opt/mssql-tools/bin"

# ===========================
# EXPORT DATABASE CREDENTIALS
# ===========================
# These environment variables are expected to already be set externally.
# Re-exporting them ensures they are available to subsequent commands.
#   DBUSER     = SQL Server username
#   DBPASSWORD = SQL Server password
#   DBENDPOINT = SQL Server host or IP with optional port (e.g., "10.0.0.3" or "10.0.0.3,1433")
export DBUSER="${DBUSER}"
export DBPASSWORD="${DBPASSWORD}"
export DBENDPOINT="${DBENDPOINT}"

# ===========================
# EXECUTE SQL FILES IN ORDER
# ===========================
# Each sqlcmd call:
#   -S : Server hostname or IP address.
#   -U : SQL Server username.
#   -P : SQL Server password.
#   -i : Input SQL script file.
# Output from each run is redirected to a separate log file under /tmp for troubleshooting.

cd /tmp
git clone https://github.com/mamonaco1973/pagila-sqlserver.git
cd pagila-sqlserver

sqlcmd -S "$DBENDPOINT" -U "$DBUSER" -P "$DBPASSWORD" -i "1.pagila-create-db.sql"   > /tmp/1.pagila-create-db.sql.log
sqlcmd -S "$DBENDPOINT" -U "$DBUSER" -P "$DBPASSWORD" -d pagila -i "2.pagila-tables.sql"      > /tmp/2.pagila-tables.sql.log
sqlcmd -S "$DBENDPOINT" -U "$DBUSER" -P "$DBPASSWORD" -d pagila -i "3.pagila-data.sql"        > /tmp/3.pagila-data.sql.log
sqlcmd -S "$DBENDPOINT" -U "$DBUSER" -P "$DBPASSWORD" -d pagila -i "4.pagila-schema.sql"      > /tmp/4.pagila-schema.sql.log


```