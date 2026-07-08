This implementaion of file system is developed by ELM Chan
https://elm-chan.org/fsw/ff/.  

This is based on FAT file system specification.  It provides
an interface to physical storage management through its disk io
API in diskio.c. The disk io interfaces are updated to interface
with Zephyr flash memory management.

Current revision in the tree is 0.16

URL to current version: https://elm-chan.org/fsw/ff/arc/ff16.zip
SHA256 20b07c8047b4976118f6488e9cd8c4b1126e0525479828986adca69bb31047d1

Security state of the FatFs sources in this tree
------------------------------------------------

In 2026 runZero disclosed a set of FatFs vulnerabilities (CVE-2026-6682 ..
CVE-2026-6688) affecting R0.16 as released. ff.c in this tree is the released
R0.16 with upstream's own patches applied on top, plus a small number of local
deltas. Each of the three groups below is a self-contained commit or commits,
so the local divergence stays auditable and easy to shed when re-baselining
onto a future FatFs release.

Fixed by upstream (applied verbatim, see https://elm-chan.org/fsw/ff/patches.html):

  R0.16 patch-1 (2025-09-13) is not a security fix; it lowers the minimum
  volume size to 64 sectors. It is applied only because patch-2 is published
  as a delta against a patch-1 tree.

  R0.16 patch-2 (2026-07-10) is upstream's response to three of the CVEs:

    CVE-2026-6682  GHSA-c25x-vcg6-4ccx  mount_volume(): cap the FAT size at the
                                        FAT32 maximum, so the FAT-area multiply
                                        cannot overflow a DWORD
    CVE-2026-6683  GHSA-74cg-p8q3-4c3q  mount_volume(): reject an exFAT volume
                                        claiming fewer than MIN_EXFAT clusters,
                                        which is what drove sync_fs() into a
                                        divide-by-zero (exFAT only, off by
                                        default)
    CVE-2026-6687  GHSA-wghw-hg6m-7rh3  f_getlabel(): clamp the exFAT volume
                                        label to the 11-character maximum
                                        (exFAT only, off by default)

Fixed locally (not addressed upstream):

    CVE-2026-6686  GHSA-h94p-vjqx-v7rr  f_lseek(): zero the region exposed when
                                        a file is extended past EOF, so a later
                                        f_read() cannot disclose residual
                                        on-disk data

  Upstream documents the expanded area as undefined and has not treated this as
  a defect, so this fix is expected to remain a local delta. It costs an
  O(size) zeroing write on an extending seek.

Not a defect, but touched for clarity:

    CVE-2026-6685  GHSA-rggj-rv54-4gvx  f_read()/f_write(): the dirty
                                        sector-cache range check

  This CVE reports the unsigned subtraction in `cached_sect - sect < cc` as an
  integer underflow. The wrap is real but benign: it is the usual
  single-comparison range check, and a cached sector before the run wraps to a
  value that is not less than cc, so the branch correctly does not fire. This
  holds for both the 32-bit and 64-bit LBA_t. Upstream made no change here in
  patch-2. We carry only a readability change that spells out the lower bound
  explicitly; behaviour is unchanged and no fix is needed.

Not carried: CVE-2026-6684 (GPT partition loop) is already bounded in R0.16;
CVE-2026-6688 is a downstream-caller pattern addressed in the Zephyr tree, not
in ff.c.
