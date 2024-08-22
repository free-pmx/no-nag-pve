# Proxmox Virtual Environment (PVE 8.2)
## No valid subscription popup removal

### TL;DR [below](#equal-single-command-alternative).

This is to assist countless people (not only) on the official Proxmox forum [1] asking about the same ("You do not have a valid subscription for this server.") and is published under AGPL [2] as is the original Proxmox Virtual Environment (PVE).

_It was published here as a reaction to having the contributing back rewarded with censorship on the official Proxmox forum._

Originally, it was also meant to protect an average PVE user from applying intricate unvetted patches of unknown origin into their JS codebase - and the reputational risk for Proxmox from assosiated security risks. Now, due to the censorship, it became yet another such patch, however hopefully much more straightforward than some others.

Finally, it is also meant to promote better coding style.

---

_Valid and tested as of version 8.2.4. Preferably apply via local console or SSH, clear browser cache afterwards._

#### Patchfile (save locally into a `$PATCHFILE` e.g. `pve8.2.4-no-nag.patch` for re-use):

```diff
--- proxmoxlib.js
+++ proxmoxlib.no-nag.js
@@ -560,23 +560,7 @@
         },
         success: function(response, opts) {
             let res = response.result;
-            if (res === null || res === undefined || !res || res
-            .data.status.toLowerCase() !== 'active') {
-            Ext.Msg.show({
-                title: gettext('No valid subscription'),
-                icon: Ext.Msg.WARNING,
-                message: Proxmox.Utils.getNoSubKeyHtml(res.data.url),
-                buttons: Ext.Msg.OK,
-                callback: function(btn) {
-                if (btn !== 'ok') {
-                    return;
-                }
-                orig_cmd();
-                },
-            });
-            } else {
             orig_cmd();
-            }
         },
         },
     );
```

#### Instructions:

Copy the `$PATCHFILE` over from your local machine onto the `$NODE`:
```sh
scp pve8.2.4-no-nag.patch root@$NODE:/usr/share/javascript/proxmox-widget-toolkit/$PATCHFILE
```

(Alternatively, you may copy & paste the content in-place into such file directly on the `$NODE`.)

Following which, install the `patch` utility, create a backup, apply the patch and restart `pveproxy` service.

```sh
apt install patch
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js{,.bak}
patch -u proxmoxlib.js -i $PATCHFILE

# the following command will disrupt your GUI access
systemctl restart pveproxy
```

#### NOTE: This is rather rudimentary and intentionally explicit approach and will not survive updates of the component, however it does not add any dubious `postinst` hooks. It also does not e.g. set your `apt` repos to `no-subscription`.

---

### Equal single-command alternative 
_proxmox-ve: 8.2.0, proxmox-widget-toolkit: 4.2.3_

```sh
(cd /usr/share/javascript/proxmox-widget-toolkit/ &&
 sha256sum -c <<< "fecc2dbc3a458442186965f0087711aecf519f797207c4dd891806ccba3636f3 proxmoxlib.js" &&
 sed -i.bak '579d;563,577d' proxmoxlib.js &&
 systemctl reload-or-restart pveproxy &&
 echo DONE - CLEAR YOUR BROWSER CACHE)
```
This checks for the file content to be exactly same as to which the patch would have been made and then deletes the same lines in-place.

You can help yourself examining the command via explainshell [3].


NOTE: If you break anything, you will need to: `apt reinstall proxmox-widget-toolkit`

[1]: https://forum.proxmox.com
[2]: https://www.gnu.org/licenses/agpl-3.0.md
[3]: https://explainshell.com/explain?cmd=%28cd+%2Fusr%2Fshare%2Fjavascript%2Fproxmox-widget-toolkit%2F+%26%26++sha256sum+-c+%3C%3C%3C+%22fecc2dbc3a458442186965f0087711aecf519f797207c4dd891806ccba3636f3+proxmoxlib.js%22+%26%26++sed+-i.bak+%27579d%3B563%2C577d%27+proxmoxlib.js+%26%26+systemctl+reload-or-restart+pveproxy+%26%26++echo+DONE+-+CLEAR+YOUR+BROWSER+CACHE%29
