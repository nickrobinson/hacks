JUNKFILES = .bash_history .irb_history .pip .ScanSnap .nbems .fldigi .cpan
JUNKFILES += .gdb_history .mysql_history .sqlite_history
BREWPACKAGES := mosh duplicity pv offlineimap wget syncthing tmux watchman
BREWCASKPACKAGES := iterm2 spotify postman
YYYYMM := $(shell date +%Y%m)
YYYYMMDD := $(shell date +%Y%m%d)

NO_COLOR = \033[0m
O1_COLOR = \033[0;01m
O2_COLOR = \033[32;01m

PREFIX = "$(O2_COLOR)==>$(O1_COLOR)"
SUFFIX = "$(NO_COLOR)"

default: routine

# FIXME make this do a local imap download too
routine: clean databackup

backup: clean mailoffsite databackup

dvbackup:
	@echo $(PREFIX) $@ $(SUFFIX)
	cd ~/Documents/datavibe/backup && make

imapbackup:
	offlineimap
	rsync -e "ssh -o Compression=no -x" \
		-avPhzy --delete-after sneak@datavibe.net:.maildir/ \
		$(HOME)/Documents/Archival/mail/sneak.datavibe.net.maildir/

mailoffsite: imapbackup
	rsync -e "ssh -o Compression=no -x" \
		-avPhzy --delete-after $(HOME)/Documents/Archival/mail/ \
		sneak@datavibe.net:.mailbackup/
	tar -xvf $(HOME)/Documents/Archival/mail/jp.eeqj.com | gzip > \
		$(HOME)/Documents/Dropbox/eeqj/archives/mail.tgz.new && \
		mv $(HOME)/Documents/Dropbox/eeqj/archives/mail.tgz.new \
			$(HOME)/Documents/Dropbox/eeqj/archives/mail.tgz

wikibackup:
	mkdir -p $(HOME)/Documents/backup/$(YYYYMMDD)
	zip -X -r $(HOME)/Documents/backup/$(YYYYMMDD)/archive.zip $(HOME)/Sync/wiki-data
	aws s3 cp $(HOME)/Documents/backup/$(YYYYMMDD)/archive.zip s3://nickrobi-documents/backup/$(YYYYMMDD)/archive.zip

cleanup:
	-mkdir -p $(HOME)/Documents/$(YYYYMM)
	-mv $(HOME)/Desktop/* $(HOME)/Documents/$(YYYYMM)

clean: cleanup
	@echo $(PREFIX) $@ $(SUFFIX)
	@-rm -rf ~/.tmp/*
	@-rm -rf ~/.Trash/*
	@-rm -rf $(JUNKFILES)

size:
	du -sh $(HOME)

lifeboat:
	mkdir -p $(HOME)/tmp/lifeboat.$(YYYYMM)
	rsync -avP --exclude='*.pkg' $(HOME)/Documents/Secure/ \
		$(HOME)/tmp/lifeboat.$(YYYYMM)/Secure/
	rsync -avP $(HOME)/Library/ApplicationSupport/Bitcoin/wallet.dat \
		$(HOME)/tmp/lifeboat.$(YYYYMM)/wallet.dat
	tar -c $(HOME)/tmp/lifeboat.$(YYYYMM) | bzip2 | \
		gpg --symmetric -a -o $(HOME)/lifeboat.$(YYYYMM).gpg
	rm -rf $(HOME)/tmp/lifeboat.$(YYYYMM)
	cp $(HOME)/lifeboat.$(YYYYMM).gpg \
		$(HOME)/dev/eeqjcdn/sneak.datavibe.net/lifeboat/lifeboat.gpg
	cd $(HOME)/dev/eeqjcdn && make
	mv $(HOME)/lifeboat.$(YYYYMM).gpg $(HOME)/Documents/Dropbox/Backups/

verify:
	duplicity verify --exclude-globbing-filelist \
		$(HOME)/.local/etc/duplicity.exclude \
		file:///Volumes/EXTUSB01/dup/ ~

remotebackup:
	RBACKUPDEST="scp://jp.eeqj.de/backup" $(HOME)/.local/bin/backup.command

packages:
		brew install $(BREWPACKAGES)

cask:
		brew cask install $(BREWCASKPACKAGES)

