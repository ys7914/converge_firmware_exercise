# Converge Hardware Exercise

Welcome to the Converge hardware programming exercise!  

## Problem Overview

Converge collects data from a wide variety of environments, from tunneling projects in the heart of London to offshore wind turbines.  Due to the mission critical nature of this data, we cannot afford to miss data when network infrastructure fails.  Whether it's the sensor being covered in concrete or a failure of the cellular backhaul network, it's critical that sensors store data that cannot be sent up to the platform locally until a connection to the server returns.   We have mounted a small flash memory IC on our nodes to act as a local, persistent data store.  Data that cannot be sent to the server will be written to the flash memory, when a connection returns the cached data will be sent up in reverse chronological order until the cache has been cleared.

In this exercise we are asking you to implement a small part of this data caching system.  In order to persist it's state across unexpected restarts, the system must write a small amount of state data to the flash memory in a separate location from the main data.  You should implement the `write_meta_data(fs_state_t* state)` function to take a structure containing state data for the system and save it to the flash memory and the `read_meta_data(fs_state_t* state)` function to search for existing state data on the flash and restore it.

The solution should take into account:

- Proper error and data integrity checking on any data written to/read from the flash
- Opening and closing the flash before reading or writing to it
- Not attempting to use the flash if it is in use by another operation
- Wear leveling on the flash memory


## Deliverables

Your solution should implement these two APIs from `fs_exercise.h` in `fs_exercise.c`.  All relevant type definitions and APIs are located in `fs_exercise.h`.  Feel free to also include any additional functions or variables which are needed for your solution.  


```
/**
* \def fs_status_t write_meta_data(fs_state_t* state)
* \brief Writes the given state to persistent flash memory
* \param state A pointer to the state structure which should be written to flash
*
* \return fs_status_t FS_OK if it wrote successfully, FS_ERROR if not, FS_BUSY if busy
*
*/
fs_status_t write_meta_data(fs_state_t* state);

/**
* \def fs_status_t read_meta_data(fs_state_t* state)
* \brief Read the latest meta data state from flash if it exists, and store it in the given structure.  Usually called on system startup to restore any state that existed before shutdown.
* \param state A pointer to the state structure which should be filled from flash data
*
* \return fs_status_t FS_OK if it read successfully, FS_ERROR if not, FS_NO_METADATA if
*                     the operation was successful but no metadata was found.
*
*/
fs_status_t read_meta_data(fs_state_t* state);

```

## Resources

### Flash memory structure

The flash memory IC consists of `FLASH_SIZE` bytes of storage split up into  `SECTOR_COUNT` sectors of `SECTOR_SIZE` bytes each.  The storage area for the main data queue takes up all but the first 3 sectors of the flash memory.  The first 2 sectors are reserved for other uses, leaving the 3rd sector free for storing the state data.

For the purposes of this exercise, assume:

```
#define   FLASH_SIZE     2048000     //bytes
#define   SECTOR_SIZE    4096        //bytes
#define   SECTOR_COUNT   (FLASH_SIZE/SECTOR_SIZE)
```


### APIs for accessing the flash memory

```
/**
* \def fs_status_t kanto_fs_open(void)
* \brief Opens the flash driver for writing, if it's not already open
*
* \return fs_status_t FS_OK if it opened, FS_ERROR if not
*
*/
fs_status_t fs_open(void);

/**
* \def fs_status_t kanto_fs_close(void)
* \brief Closes, if it's open
*
* \return fs_status_t FS_OK if it closed, FS_ERROR if not
*
*/
fs_status_t fs_close(void);

/**
* \def fs_status_t kanto_fs_flash_is_formatting(void)
* \brief Checks if the flash chip is busy being formatted, returns immediatly
*
* \return fs_status_t FS_OK if it's not busy, FS_BUSY if it is
*
*/
fs_status_t fs_flash_is_formatting(void);

/**
 * \brief Read storage content
 * \param offset Address to read from
 * \param length Number of bytes to read
 * \param buf Buffer where to store the read bytes
 * \return True when successful.
 *
 * buf must be allocated by the caller
 */
bool fs_flash_read(size_t offset, size_t length, uint8_t *buf);

/**
 * \brief Write to storage sectors.
 * \param offset Address to write to
 * \param length Number of bytes to write
 * \param buf Buffer holding the bytes to be written
 *
 * \return True when successful.
 */
bool fs_flash_write(size_t offset, size_t length, const uint8_t *buf);

/**
 * \brief Erase storage sectors corresponding to the range.
 * \param offset Address to start erasing
 * \param length Number of bytes to erase
 * \return True when successful.
 *
 * The erase operation will be sector-wise, therefore a call to this function
 * will generally start the erase procedure at an address lower than offset
 */
bool fs_flash_erase(size_t offset, size_t length);

/**
 * \brief Check if the flash is busy or not
 *
 * \return True if the flash is free to use, false if busy
 */
bool fs_flash_is_ready(void);
```

### Other useful functions
```
/**
 * \brief      Calculate the CRC16 over a data area
 * \param data Pointer to the data
 * \param datalen The length of the data
 * \param acc  The accumulated CRC that is to be updated (or zero).
 * \return     The CRC16 checksum.
 *
 *             This function calculates the CRC16 checksum of a data area.
 */
unsigned short crc16_data(const unsigned char *data, int datalen,
			  unsigned short acc);
```
