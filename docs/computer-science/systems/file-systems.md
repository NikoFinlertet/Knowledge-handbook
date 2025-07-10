# Файловые системы 📁

> **Навигация**: [[README|← Системное программирование]] | [[computer-science/README|Компьютерные науки]] | [[README|Главная]]

## 🎯 Цель изучения

Глубокое понимание файловых систем, их архитектуры, оптимизации производительности и программирования файлового I/O на системном уровне.

## 📚 Теоретические основы

### 🏗️ Архитектура файловых систем

#### Базовые концепции
```c
// Основные компоненты файловой системы:

// 1. Superblock - метаданные файловой системы
struct superblock {
    uint32_t magic;           // Магическое число ФС
    uint32_t block_size;      // Размер блока
    uint32_t total_blocks;    // Общее количество блоков
    uint32_t free_blocks;     // Свободные блоки
    uint32_t total_inodes;    // Общее количество inode
    uint32_t free_inodes;     // Свободные inode
    uint32_t root_inode;      // Корневая директория
    time_t   mount_time;      // Время монтирования
    time_t   write_time;      // Последняя запись
};

// 2. Inode - информация о файле
struct inode {
    uint16_t mode;            // Права доступа и тип файла
    uint16_t uid;             // ID пользователя
    uint16_t gid;             // ID группы
    uint32_t size;            // Размер файла
    time_t   atime;           // Время доступа
    time_t   mtime;           // Время модификации
    time_t   ctime;           // Время изменения inode
    uint32_t links_count;     // Количество жестких ссылок
    uint32_t blocks[15];      // Указатели на блоки данных
    // blocks[0-11] - прямые блоки
    // blocks[12] - одинарная косвенность
    // blocks[13] - двойная косвенность  
    // blocks[14] - тройная косвенность
};

// 3. Directory entry - запись в директории
struct dirent {
    uint32_t inode_num;       // Номер inode
    uint16_t rec_len;         // Длина записи
    uint8_t  name_len;        // Длина имени
    uint8_t  file_type;       // Тип файла
    char     name[];          // Имя файла (переменная длина)
};
```

#### Типы файловых систем
```c
// Классификация по архитектуре:

// 1. Дисковые ФС
// ext4 - Linux extended filesystem
// NTFS - Windows NT File System
// APFS - Apple File System
// ZFS  - Zettabyte File System

// 2. Сетевые ФС
// NFS  - Network File System
// SMB/CIFS - Server Message Block
// AFS  - Andrew File System

// 3. Виртуальные ФС
// proc - информация о процессах
// sys  - информация о системе
// dev  - устройства
// tmp  - временные файлы

// 4. Специализированные ФС
// FAT32 - File Allocation Table
// ISO9660 - CD-ROM
// FUSE - Filesystem in Userspace
```

### 📂 Файловый I/O в системном программировании

#### Низкоуровневые системные вызовы
```c
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

// Основные файловые операции
int file_operations_demo() {
    int fd;
    char buffer[1024];
    ssize_t bytes_read, bytes_written;
    struct stat file_stat;
    
    // Открытие файла
    fd = open("example.txt", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("open");
        return -1;
    }
    
    // Запись данных
    const char* data = "Hello, File System!\n";
    bytes_written = write(fd, data, strlen(data));
    if (bytes_written == -1) {
        perror("write");
        close(fd);
        return -1;
    }
    
    // Перемещение к началу файла
    if (lseek(fd, 0, SEEK_SET) == -1) {
        perror("lseek");
        close(fd);
        return -1;
    }
    
    // Чтение данных
    bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("read");
        close(fd);
        return -1;
    }
    buffer[bytes_read] = '\0';
    
    printf("Read: %s", buffer);
    
    // Получение информации о файле
    if (fstat(fd, &file_stat) == -1) {
        perror("fstat");
        close(fd);
        return -1;
    }
    
    printf("File size: %ld bytes\n", file_stat.st_size);
    printf("File permissions: %o\n", file_stat.st_mode & 0777);
    printf("Last modified: %s", ctime(&file_stat.st_mtime));
    
    // Синхронизация данных с диском
    if (fsync(fd) == -1) {
        perror("fsync");
    }
    
    close(fd);
    return 0;
}

// Работа с флагами открытия
void open_flags_demo() {
    int fd;
    
    // Создание файла с эксклюзивным доступом
    fd = open("exclusive.txt", O_CREAT | O_EXCL | O_WRONLY, 0644);
    if (fd == -1) {
        if (errno == EEXIST) {
            printf("File already exists\n");
        }
        return;
    }
    
    // Неблокирующий открытие (для FIFO/pipes)
    fd = open("/tmp/mypipe", O_RDONLY | O_NONBLOCK);
    if (fd == -1) {
        if (errno == EAGAIN) {
            printf("Would block - no writers\n");
        }
    }
    
    // Открытие с sync - каждая запись синхронизируется
    fd = open("sync_file.txt", O_WRONLY | O_CREAT | O_SYNC, 0644);
    // Все write() будут ждать физической записи на диск
    
    close(fd);
}
```

#### Продвинутые файловые операции
```c
#include <sys/mman.h>
#include <sys/sendfile.h>

// Memory-mapped файлы
void memory_mapped_file_demo() {
    int fd;
    size_t file_size;
    char* mapped_data;
    struct stat sb;
    
    fd = open("large_file.dat", O_RDWR);
    if (fd == -1) {
        perror("open");
        return;
    }
    
    // Получение размера файла
    if (fstat(fd, &sb) == -1) {
        perror("fstat");
        close(fd);
        return;
    }
    file_size = sb.st_size;
    
    // Отображение файла в память
    mapped_data = mmap(NULL, file_size, PROT_READ | PROT_WRITE,
                      MAP_SHARED, fd, 0);
    if (mapped_data == MAP_FAILED) {
        perror("mmap");
        close(fd);
        return;
    }
    
    // Работа с файлом как с обычной памятью
    mapped_data[0] = 'X';
    mapped_data[file_size - 1] = 'Y';
    
    // Принудительная синхронизация с диском
    if (msync(mapped_data, file_size, MS_SYNC) == -1) {
        perror("msync");
    }
    
    // Освобождение отображения
    munmap(mapped_data, file_size);
    close(fd);
}

// Zero-copy file transfer
ssize_t copy_file_zero_copy(const char* source, const char* dest) {
    int src_fd, dest_fd;
    struct stat stat_buf;
    ssize_t total_sent = 0;
    
    src_fd = open(source, O_RDONLY);
    if (src_fd == -1) {
        perror("open source");
        return -1;
    }
    
    dest_fd = open(dest, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dest_fd == -1) {
        perror("open dest");
        close(src_fd);
        return -1;
    }
    
    if (fstat(src_fd, &stat_buf) == -1) {
        perror("fstat");
        close(src_fd);
        close(dest_fd);
        return -1;
    }
    
    // sendfile() - zero-copy transfer
    total_sent = sendfile(dest_fd, src_fd, NULL, stat_buf.st_size);
    
    close(src_fd);
    close(dest_fd);
    
    return total_sent;
}

// Асинхронный I/O
#include <aio.h>

void async_file_operations() {
    struct aiocb aio_req;
    int fd;
    char buffer[1024];
    int ret;
    
    fd = open("async_file.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return;
    }
    
    // Настройка асинхронного запроса
    memset(&aio_req, 0, sizeof(aio_req));
    aio_req.aio_fildes = fd;
    aio_req.aio_buf = buffer;
    aio_req.aio_nbytes = sizeof(buffer);
    aio_req.aio_offset = 0;
    
    // Запуск асинхронного чтения
    if (aio_read(&aio_req) == -1) {
        perror("aio_read");
        close(fd);
        return;
    }
    
    // Можем делать другую работу...
    printf("Doing other work while reading...\n");
    
    // Ожидание завершения операции
    while (aio_error(&aio_req) == EINPROGRESS) {
        usleep(1000); // 1ms
    }
    
    ret = aio_return(&aio_req);
    if (ret > 0) {
        printf("Read %d bytes asynchronously\n", ret);
        buffer[ret] = '\0';
        printf("Data: %s\n", buffer);
    }
    
    close(fd);
}
```

### 📊 Буферизация и кэширование

#### Системы буферизации
```c
#include <stdio.h>

// Настройка буферизации stdio
void buffering_demo() {
    FILE* fp;
    char buffer[8192];
    
    fp = fopen("buffered_file.txt", "w");
    if (fp == NULL) {
        perror("fopen");
        return;
    }
    
    // Установка полной буферизации
    setvbuf(fp, buffer, _IOFBF, sizeof(buffer));
    
    // Запись будет буферизована
    fprintf(fp, "This data is buffered\n");
    
    // Принудительная запись буфера
    fflush(fp);
    
    fclose(fp);
    
    // Небуферизованный вывод
    fp = fopen("unbuffered_file.txt", "w");
    setvbuf(fp, NULL, _IONBF, 0);
    
    fprintf(fp, "This data is written immediately\n");
    
    fclose(fp);
}

// Управление системным кэшем
void cache_management() {
    int fd;
    
    fd = open("cache_test.txt", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("open");
        return;
    }
    
    // Указание о последовательном доступе
    posix_fadvise(fd, 0, 0, POSIX_FADV_SEQUENTIAL);
    
    // Указание о случайном доступе
    posix_fadvise(fd, 0, 0, POSIX_FADV_RANDOM);
    
    // Предварительная загрузка в кэш
    posix_fadvise(fd, 0, 1024*1024, POSIX_FADV_WILLNEED);
    
    // Указание что данные больше не нужны
    posix_fadvise(fd, 0, 1024*1024, POSIX_FADV_DONTNEED);
    
    close(fd);
}

// Direct I/O (обход кэша)
void direct_io_demo() {
    int fd;
    void* aligned_buffer;
    ssize_t bytes_written;
    
    // Открытие с флагом O_DIRECT
    fd = open("direct_io_file.txt", O_WRONLY | O_CREAT | O_DIRECT, 0644);
    if (fd == -1) {
        perror("open with O_DIRECT");
        return;
    }
    
    // Буфер должен быть выровнен по границе сектора
    if (posix_memalign(&aligned_buffer, 512, 4096) != 0) {
        perror("posix_memalign");
        close(fd);
        return;
    }
    
    memset(aligned_buffer, 'A', 4096);
    
    // Запись напрямую на диск (обход кэша)
    bytes_written = write(fd, aligned_buffer, 4096);
    if (bytes_written == -1) {
        perror("write");
    }
    
    free(aligned_buffer);
    close(fd);
}
```

## 🔍 Мониторинг файловых систем

### 📈 Мониторинг файловых событий (inotify)
```c
#include <sys/inotify.h>

#define EVENT_SIZE (sizeof(struct inotify_event))
#define BUF_LEN (1024 * (EVENT_SIZE + 16))

// Мониторинг изменений в файловой системе
void file_system_monitor(const char* path) {
    int fd, wd;
    char buffer[BUF_LEN];
    
    // Создание inotify instance
    fd = inotify_init();
    if (fd < 0) {
        perror("inotify_init");
        return;
    }
    
    // Добавление watch для директории
    wd = inotify_add_watch(fd, path, 
                          IN_CREATE | IN_DELETE | IN_MODIFY | 
                          IN_MOVED_FROM | IN_MOVED_TO);
    if (wd < 0) {
        perror("inotify_add_watch");
        close(fd);
        return;
    }
    
    printf("Monitoring directory: %s\n", path);
    
    while (1) {
        int length = read(fd, buffer, BUF_LEN);
        if (length < 0) {
            perror("read");
            break;
        }
        
        int i = 0;
        while (i < length) {
            struct inotify_event* event = (struct inotify_event*)&buffer[i];
            
            printf("Event: ");
            if (event->mask & IN_CREATE)
                printf("File/Directory created: %s\n", event->name);
            if (event->mask & IN_DELETE)
                printf("File/Directory deleted: %s\n", event->name);
            if (event->mask & IN_MODIFY)
                printf("File modified: %s\n", event->name);
            if (event->mask & IN_MOVED_FROM)
                printf("File moved from: %s\n", event->name);
            if (event->mask & IN_MOVED_TO)
                printf("File moved to: %s\n", event->name);
            
            i += EVENT_SIZE + event->len;
        }
    }
    
    inotify_rm_watch(fd, wd);
    close(fd);
}

// Рекурсивный мониторинг директорий
typedef struct watch_node {
    int wd;
    char path[PATH_MAX];
    struct watch_node* next;
} watch_node_t;

static watch_node_t* watch_list = NULL;

void add_watch_recursive(int inotify_fd, const char* path) {
    DIR* dir;
    struct dirent* entry;
    char full_path[PATH_MAX];
    int wd;
    
    // Добавляем watch для текущей директории
    wd = inotify_add_watch(inotify_fd, path,
                          IN_CREATE | IN_DELETE | IN_MODIFY | 
                          IN_MOVED_FROM | IN_MOVED_TO | IN_ISDIR);
    
    if (wd >= 0) {
        watch_node_t* node = malloc(sizeof(watch_node_t));
        node->wd = wd;
        strncpy(node->path, path, PATH_MAX - 1);
        node->next = watch_list;
        watch_list = node;
        
        printf("Added watch for: %s (wd=%d)\n", path, wd);
    }
    
    // Рекурсивно добавляем watch для поддиректорий
    dir = opendir(path);
    if (dir == NULL) {
        return;
    }
    
    while ((entry = readdir(dir)) != NULL) {
        if (entry->d_type == DT_DIR && 
            strcmp(entry->d_name, ".") != 0 && 
            strcmp(entry->d_name, "..") != 0) {
            
            snprintf(full_path, PATH_MAX, "%s/%s", path, entry->d_name);
            add_watch_recursive(inotify_fd, full_path);
        }
    }
    
    closedir(dir);
}
```

### 📊 Анализ производительности файловой системы
```c
#include <sys/statvfs.h>
#include <time.h>

// Получение статистики файловой системы
void filesystem_stats(const char* path) {
    struct statvfs fs_stat;
    
    if (statvfs(path, &fs_stat) != 0) {
        perror("statvfs");
        return;
    }
    
    printf("=== Filesystem Statistics ===\n");
    printf("Path: %s\n", path);
    printf("Block size: %lu bytes\n", fs_stat.f_bsize);
    printf("Fragment size: %lu bytes\n", fs_stat.f_frsize);
    printf("Total blocks: %lu\n", fs_stat.f_blocks);
    printf("Free blocks: %lu\n", fs_stat.f_bfree);
    printf("Available blocks: %lu\n", fs_stat.f_bavail);
    printf("Total inodes: %lu\n", fs_stat.f_files);
    printf("Free inodes: %lu\n", fs_stat.f_ffree);
    printf("Available inodes: %lu\n", fs_stat.f_favail);
    
    // Вычисление размеров
    unsigned long total_size = fs_stat.f_blocks * fs_stat.f_frsize;
    unsigned long free_size = fs_stat.f_bavail * fs_stat.f_frsize;
    unsigned long used_size = total_size - free_size;
    
    printf("\nSize Information:\n");
    printf("Total size: %.2f GB\n", total_size / (1024.0 * 1024.0 * 1024.0));
    printf("Used size: %.2f GB\n", used_size / (1024.0 * 1024.0 * 1024.0));
    printf("Free size: %.2f GB\n", free_size / (1024.0 * 1024.0 * 1024.0));
    printf("Usage: %.1f%%\n", (used_size * 100.0) / total_size);
}

// Бенчмарк производительности I/O
typedef struct {
    double read_time;
    double write_time;
    double sync_time;
    size_t bytes_transferred;
    double throughput_mb_s;
} io_benchmark_t;

io_benchmark_t benchmark_file_io(const char* filename, size_t file_size) {
    io_benchmark_t result = {0};
    int fd;
    char* buffer;
    struct timespec start, end;
    
    // Подготовка буфера
    buffer = malloc(file_size);
    if (buffer == NULL) {
        perror("malloc");
        return result;
    }
    memset(buffer, 'A', file_size);
    
    // === WRITE BENCHMARK ===
    fd = open(filename, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open for write");
        free(buffer);
        return result;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &start);
    ssize_t written = write(fd, buffer, file_size);
    clock_gettime(CLOCK_MONOTONIC, &end);
    
    result.write_time = (end.tv_sec - start.tv_sec) + 
                       (end.tv_nsec - start.tv_nsec) / 1e9;
    
    // Sync benchmark
    clock_gettime(CLOCK_MONOTONIC, &start);
    fsync(fd);
    clock_gettime(CLOCK_MONOTONIC, &end);
    
    result.sync_time = (end.tv_sec - start.tv_sec) + 
                      (end.tv_nsec - start.tv_nsec) / 1e9;
    
    close(fd);
    
    // === READ BENCHMARK ===
    fd = open(filename, O_RDONLY);
    if (fd == -1) {
        perror("open for read");
        free(buffer);
        return result;
    }
    
    clock_gettime(CLOCK_MONOTONIC, &start);
    ssize_t read_bytes = read(fd, buffer, file_size);
    clock_gettime(CLOCK_MONOTONIC, &end);
    
    result.read_time = (end.tv_sec - start.tv_sec) + 
                      (end.tv_nsec - start.tv_nsec) / 1e9;
    
    close(fd);
    free(buffer);
    
    // Вычисление результатов
    result.bytes_transferred = file_size;
    double total_time = result.read_time + result.write_time;
    result.throughput_mb_s = (file_size * 2.0) / (1024.0 * 1024.0) / total_time;
    
    return result;
}

void print_benchmark_results(const io_benchmark_t* result) {
    printf("=== I/O Benchmark Results ===\n");
    printf("Bytes transferred: %zu\n", result->bytes_transferred);
    printf("Write time: %.3f seconds\n", result->write_time);
    printf("Read time: %.3f seconds\n", result->read_time);
    printf("Sync time: %.3f seconds\n", result->sync_time);
    printf("Overall throughput: %.2f MB/s\n", result->throughput_mb_s);
    printf("Write speed: %.2f MB/s\n", 
           (result->bytes_transferred / (1024.0 * 1024.0)) / result->write_time);
    printf("Read speed: %.2f MB/s\n", 
           (result->bytes_transferred / (1024.0 * 1024.0)) / result->read_time);
}
```

## 🗃️ Специализированные файловые системы

### 💾 Работа с ext4
```c
#include <ext2fs/ext2_fs.h>

// Информация специфичная для ext4
void ext4_specific_info(const char* device) {
    int fd;
    struct ext2_super_block sb;
    
    fd = open(device, O_RDONLY);
    if (fd == -1) {
        perror("open device");
        return;
    }
    
    // Чтение суперблока (смещение 1024 байта)
    if (lseek(fd, 1024, SEEK_SET) == -1) {
        perror("lseek");
        close(fd);
        return;
    }
    
    if (read(fd, &sb, sizeof(sb)) != sizeof(sb)) {
        perror("read superblock");
        close(fd);
        return;
    }
    
    // Проверка магического числа ext4
    if (sb.s_magic != EXT2_SUPER_MAGIC) {
        printf("Not an ext2/3/4 filesystem\n");
        close(fd);
        return;
    }
    
    printf("=== EXT4 Filesystem Information ===\n");
    printf("Block size: %d bytes\n", 1024 << sb.s_log_block_size);
    printf("Total blocks: %u\n", sb.s_blocks_count);
    printf("Free blocks: %u\n", sb.s_free_blocks_count);
    printf("Total inodes: %u\n", sb.s_inodes_count);
    printf("Free inodes: %u\n", sb.s_free_inodes_count);
    printf("Filesystem version: %u.%u\n", sb.s_rev_level, sb.s_minor_rev_level);
    
    close(fd);
}

// Работа с extended attributes
#include <sys/xattr.h>

void extended_attributes_demo(const char* filename) {
    const char* attr_name = "user.description";
    const char* attr_value = "This is a test file with extended attributes";
    char buffer[256];
    ssize_t attr_size;
    
    // Установка extended attribute
    if (setxattr(filename, attr_name, attr_value, strlen(attr_value), 0) == -1) {
        perror("setxattr");
        return;
    }
    
    // Получение размера атрибута
    attr_size = getxattr(filename, attr_name, NULL, 0);
    if (attr_size == -1) {
        perror("getxattr size");
        return;
    }
    
    // Чтение атрибута
    if (getxattr(filename, attr_name, buffer, attr_size) == -1) {
        perror("getxattr");
        return;
    }
    
    buffer[attr_size] = '\0';
    printf("Extended attribute '%s': %s\n", attr_name, buffer);
    
    // Список всех атрибутов
    char attr_list[1024];
    ssize_t list_size = listxattr(filename, attr_list, sizeof(attr_list));
    if (list_size > 0) {
        printf("All extended attributes:\n");
        char* attr = attr_list;
        while (attr < attr_list + list_size) {
            printf("  %s\n", attr);
            attr += strlen(attr) + 1;
        }
    }
    
    // Удаление атрибута
    if (removexattr(filename, attr_name) == -1) {
        perror("removexattr");
    }
}
```

### 🔗 FUSE (Filesystem in Userspace)
```c
#define FUSE_USE_VERSION 31
#include <fuse.h>

// Простая FUSE файловая система в памяти
struct memory_file {
    char name[256];
    char* data;
    size_t size;
    mode_t mode;
    time_t atime, mtime, ctime;
};

static struct memory_file files[100];
static int file_count = 0;

// Поиск файла по имени
static struct memory_file* find_file(const char* path) {
    for (int i = 0; i < file_count; i++) {
        if (strcmp(files[i].name, path) == 0) {
            return &files[i];
        }
    }
    return NULL;
}

// FUSE операция: получение атрибутов файла
static int memory_getattr(const char* path, struct stat* stbuf,
                         struct fuse_file_info* fi) {
    memset(stbuf, 0, sizeof(struct stat));
    
    if (strcmp(path, "/") == 0) {
        stbuf->st_mode = S_IFDIR | 0755;
        stbuf->st_nlink = 2;
        return 0;
    }
    
    struct memory_file* file = find_file(path);
    if (file == NULL) {
        return -ENOENT;
    }
    
    stbuf->st_mode = file->mode;
    stbuf->st_nlink = 1;
    stbuf->st_size = file->size;
    stbuf->st_atime = file->atime;
    stbuf->st_mtime = file->mtime;
    stbuf->st_ctime = file->ctime;
    
    return 0;
}

// FUSE операция: чтение директории
static int memory_readdir(const char* path, void* buf, fuse_fill_dir_t filler,
                         off_t offset, struct fuse_file_info* fi,
                         enum fuse_readdir_flags flags) {
    if (strcmp(path, "/") != 0) {
        return -ENOENT;
    }
    
    filler(buf, ".", NULL, 0, 0);
    filler(buf, "..", NULL, 0, 0);
    
    for (int i = 0; i < file_count; i++) {
        filler(buf, files[i].name + 1, NULL, 0, 0); // убираем '/' в начале
    }
    
    return 0;
}

// FUSE операция: чтение файла
static int memory_read(const char* path, char* buf, size_t size, off_t offset,
                      struct fuse_file_info* fi) {
    struct memory_file* file = find_file(path);
    if (file == NULL) {
        return -ENOENT;
    }
    
    if (offset >= file->size) {
        return 0;
    }
    
    if (offset + size > file->size) {
        size = file->size - offset;
    }
    
    memcpy(buf, file->data + offset, size);
    file->atime = time(NULL);
    
    return size;
}

// FUSE операция: запись файла
static int memory_write(const char* path, const char* buf, size_t size,
                       off_t offset, struct fuse_file_info* fi) {
    struct memory_file* file = find_file(path);
    if (file == NULL) {
        return -ENOENT;
    }
    
    if (offset + size > file->size) {
        file->data = realloc(file->data, offset + size);
        file->size = offset + size;
    }
    
    memcpy(file->data + offset, buf, size);
    file->mtime = time(NULL);
    
    return size;
}

// FUSE операция: создание файла
static int memory_create(const char* path, mode_t mode,
                        struct fuse_file_info* fi) {
    if (file_count >= 100) {
        return -ENOSPC;
    }
    
    struct memory_file* file = &files[file_count++];
    strncpy(file->name, path, sizeof(file->name) - 1);
    file->data = malloc(1);
    file->size = 0;
    file->mode = S_IFREG | mode;
    file->atime = file->mtime = file->ctime = time(NULL);
    
    return 0;
}

// Таблица FUSE операций
static struct fuse_operations memory_operations = {
    .getattr = memory_getattr,
    .readdir = memory_readdir,
    .read = memory_read,
    .write = memory_write,
    .create = memory_create,
};

// Запуск FUSE файловой системы
int run_memory_filesystem(int argc, char* argv[]) {
    return fuse_main(argc, argv, &memory_operations, NULL);
}
```

## 🛠️ Практические инструменты

### 🔧 Утилиты для работы с файловыми системами
```c
#include <sys/mount.h>

// Монтирование файловой системы
int mount_filesystem(const char* device, const char* mountpoint,
                    const char* fstype, const char* options) {
    unsigned long mountflags = 0;
    
    // Парсинг опций монтирования
    if (strstr(options, "ro")) mountflags |= MS_RDONLY;
    if (strstr(options, "noexec")) mountflags |= MS_NOEXEC;
    if (strstr(options, "nosuid")) mountflags |= MS_NOSUID;
    if (strstr(options, "nodev")) mountflags |= MS_NODEV;
    if (strstr(options, "sync")) mountflags |= MS_SYNCHRONOUS;
    
    if (mount(device, mountpoint, fstype, mountflags, options) == -1) {
        perror("mount");
        return -1;
    }
    
    printf("Successfully mounted %s on %s\n", device, mountpoint);
    return 0;
}

// Размонтирование файловой системы
int unmount_filesystem(const char* mountpoint, int force) {
    int flags = 0;
    
    if (force) {
        flags |= MNT_FORCE;
    }
    
    if (umount2(mountpoint, flags) == -1) {
        perror("umount2");
        return -1;
    }
    
    printf("Successfully unmounted %s\n", mountpoint);
    return 0;
}

// Создание файловой системы
int create_ext4_filesystem(const char* device) {
    char command[512];
    
    snprintf(command, sizeof(command), "mkfs.ext4 -F %s", device);
    
    printf("Creating ext4 filesystem on %s...\n", device);
    int result = system(command);
    
    if (result == 0) {
        printf("Filesystem created successfully\n");
    } else {
        printf("Failed to create filesystem\n");
    }
    
    return result;
}

// Проверка файловой системы
int check_filesystem(const char* device) {
    char command[512];
    
    snprintf(command, sizeof(command), "fsck -y %s", device);
    
    printf("Checking filesystem on %s...\n", device);
    int result = system(command);
    
    return result;
}
```

### 📊 Дефрагментация и оптимизация
```c
// Анализ фрагментации файла
void analyze_file_fragmentation(const char* filename) {
    int fd;
    struct stat st;
    off_t file_size;
    blkcnt_t blocks_used;
    blksize_t block_size;
    
    fd = open(filename, O_RDONLY);
    if (fd == -1) {
        perror("open");
        return;
    }
    
    if (fstat(fd, &st) == -1) {
        perror("fstat");
        close(fd);
        return;
    }
    
    file_size = st.st_size;
    blocks_used = st.st_blocks;
    block_size = st.st_blksize;
    
    // Вычисление ожидаемого количества блоков
    blkcnt_t expected_blocks = (file_size + block_size - 1) / block_size;
    
    printf("=== File Fragmentation Analysis ===\n");
    printf("File: %s\n", filename);
    printf("File size: %ld bytes\n", file_size);
    printf("Block size: %ld bytes\n", block_size);
    printf("Blocks used: %ld\n", blocks_used);
    printf("Expected blocks: %ld\n", expected_blocks);
    
    if (blocks_used > expected_blocks) {
        printf("File appears to be fragmented\n");
        printf("Fragmentation ratio: %.2f\n", 
               (double)blocks_used / expected_blocks);
    } else {
        printf("File appears to be well-aligned\n");
    }
    
    close(fd);
}

// Оптимизация размещения файлов
void optimize_file_placement(const char* filename) {
    int src_fd, dst_fd;
    char temp_filename[PATH_MAX];
    char buffer[64 * 1024]; // 64KB buffer
    ssize_t bytes_read, bytes_written;
    
    // Создание временного файла
    snprintf(temp_filename, sizeof(temp_filename), "%s.tmp", filename);
    
    src_fd = open(filename, O_RDONLY);
    if (src_fd == -1) {
        perror("open source");
        return;
    }
    
    dst_fd = open(temp_filename, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dst_fd == -1) {
        perror("open temp");
        close(src_fd);
        return;
    }
    
    // Копирование с большими блоками для лучшего размещения
    while ((bytes_read = read(src_fd, buffer, sizeof(buffer))) > 0) {
        bytes_written = write(dst_fd, buffer, bytes_read);
        if (bytes_written != bytes_read) {
            perror("write");
            break;
        }
    }
    
    close(src_fd);
    close(dst_fd);
    
    // Замена оригинального файла
    if (rename(temp_filename, filename) == -1) {
        perror("rename");
        unlink(temp_filename);
    } else {
        printf("File optimized: %s\n", filename);
    }
}
```

## 🎯 Практические проекты

### 🎯 Проект 1: File System Monitor (2-3 недели)
```c
// Основные компоненты:
// 1. Мониторинг изменений в реальном времени (inotify)
// 2. Логирование файловых операций
// 3. Статистика использования диска
// 4. Уведомления о критических событиях
// 5. Web-интерфейс для просмотра логов
// 6. Конфигурационные файлы

// Пример структуры основного цикла:
typedef struct {
    int inotify_fd;
    char monitored_paths[10][PATH_MAX];
    int path_count;
    FILE* log_file;
    time_t start_time;
} fs_monitor_t;

int main() {
    fs_monitor_t monitor = {0};
    
    // Инициализация мониторинга
    setup_monitoring(&monitor);
    
    // Основной цикл обработки событий
    run_monitoring_loop(&monitor);
    
    // Очистка ресурсов
    cleanup_monitoring(&monitor);
    
    return 0;
}
```

### 🎯 Проект 2: High-Performance File Server (3-4 недели)
```c
// Функциональность:
// 1. HTTP сервер для файлов
// 2. Zero-copy передача (sendfile)
// 3. Memory-mapped файлы для кэширования
// 4. Range requests (частичная загрузка)
// 5. Gzip compression на лету
// 6. Access control и аутентификация
// 7. Bandwidth throttling
// 8. Detailed logging и метрики

// Архитектура сервера:
typedef struct {
    int server_socket;
    char document_root[PATH_MAX];
    size_t max_file_cache_size;
    hash_table_t* file_cache;
    thread_pool_t* workers;
    access_control_t* acl;
} file_server_t;
```

### 🎯 Проект 3: Backup System (4-5 недель)
```c
// Возможности:
// 1. Инкрементальные и дифференциальные бэкапы
// 2. Дедупликация данных
// 3. Сжатие архивов
// 4. Шифрование бэкапов
// 5. Восстановление до определенной точки
// 6. Планировщик задач
// 7. Проверка целостности архивов
// 8. Удаленное хранение (FTP/SFTP/S3)

// Основные структуры:
typedef struct {
    char source_path[PATH_MAX];
    char backup_path[PATH_MAX];
    backup_type_t type; // FULL, INCREMENTAL, DIFFERENTIAL
    compression_level_t compression;
    encryption_key_t encryption;
    time_t last_backup;
    checksum_t integrity_hash;
} backup_job_t;
```

## 📚 Дополнительные ресурсы

### 📖 Рекомендуемая литература
- **"Understanding the Linux Kernel" - Daniel P. Bovet** `🐧 Linux специфика`
- **"Linux System Programming" - Robert Love** `📘 Практические аспекты`
- **"File System Forensic Analysis" - Brian Carrier** `🔍 Глубокий анализ`

### 🔗 Связанные темы
- [[memory-management|Управление памятью]] - для буферизации и кэширования
- [[processes-threads|Процессы и потоки]] - для файловых серверов
- [[../technical-skills/databases/README|Базы данных]] - для индексирования файлов
- [[../infrastructure/linux-deployment|Linux развертывание]] - системное администрирование
- [[../technical-skills/security|Безопасность]] - права доступа и шифрование

### 🏗️ Roadmap развития
1. **Неделя 1**: Основы файлового I/O, системные вызовы
2. **Неделя 2**: Продвинутые операции, memory mapping, асинхронный I/O
3. **Неделя 3**: Мониторинг файловых систем, inotify, статистика
4. **Неделя 4**: Специализированные ФС, FUSE, extended attributes
5. **Неделя 5**: Оптимизация производительности, бенчмарки
6. **Неделя 6**: Практические проекты и интеграция

---

> 💡 **Совет**: Изучайте файловые системы в связке с kernel источниками Linux. Это даст глубокое понимание внутренних механизмов! 