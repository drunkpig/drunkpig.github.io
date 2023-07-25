

wget -w 1 -np -m -A download <link_to_sourceforge_folder>
grep -Rh refresh sourceforge.net | grep -o "https[^\\?]*" > urllist
 wget -P <folder_where_you_want_files_to_be_downloaded> -i urllist