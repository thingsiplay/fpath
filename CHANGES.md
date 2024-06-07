# Changelog

Keep track of changes for [fpath](https://github.com/thingsiplay/fpath)

## Next

- initial Changelog

- new: resolve versions with "r" in name of path based commands added, these
  are the same as the normal variants but will resolve a symbolic link to its
  target: `{rlist}`, `{rstart:end}`, `{rpath}`, `{rrot}`, `{ruri}`, `{rdir}`,
  `{rdirname}`, `{rname}`, `{rstem}`, `{rext}`, `{rexts}`, `{rfile}`, `{rtype}`,
  `{rmime}`, `{rowner}` and `{rgroup}`
- removed: all `{1}..{9}` single index based commands removed, as they were not
  that useful anyway and we have more flexible slice based alternative
- removed: due to technical difficulties and limited usefulness, all isvalid
  based commands are removed: `{isvalid}`, `{isnotvalid}`, `{isvalid:TEXT}` and
  `{isnotvalid:TEXT}`
- help: output of long help for `fpath -H fmt` reworked and greatly simplified
