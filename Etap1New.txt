#!/bin/bash

# Основные директории
BACKUP_DIR="/var/db/postgres2/backups"
WAL_ARCHIVE_DIR="/var/db/postgres2/wal_archive"
WAL_DIR="$HOME/qfg95"

# Резервный сервер
REMOTE_SERVER="postgres4@pg111"
REMOTE_BACKUP_DIR="/var/db/postgres4/backups"
REMOTE_WAL_DIR="/var/db/postgres4/wal_archive"

# Создание локальных директорий
mkdir -p "$BACKUP_DIR"
mkdir -p "$WAL_ARCHIVE_DIR"

# Имя бэкапа с датой
BACKUP_NAME="backup_$(date +%F_%H-%M-%S)"

# 1. Создание полной резервной копии
echo "Creating full base backup..."
pg_basebackup -D "$BACKUP_DIR/$BACKUP_NAME" -Ft -z -P -X stream -U postgres2 -h 127.0.0.1 -p 9426

# 2. Копирование полной резервной копии на резервный сервер
echo "Copying base backup to standby server..."
rsync -avz "$BACKUP_DIR/$BACKUP_NAME/" "$REMOTE_SERVER:$REMOTE_BACKUP_DIR/"

# 3. Архивирование WAL файлов на основной сервер (локально)
echo "Archiving WAL files locally..."
rsync -avzu "$WAL_DIR/"*.backup "$WAL_ARCHIVE_DIR/" 2>/dev/null
rsync -avzu "$WAL_DIR"/0000* "$WAL_ARCHIVE_DIR/" 2>/dev/null

# 4. Копирование WAL на резервный сервер
echo "Copying WAL files to standby server..."
rsync -avzu "$WAL_ARCHIVE_DIR/" "$REMOTE_SERVER:$REMOTE_WAL_DIR/"

echo "Backup procedure finished."
