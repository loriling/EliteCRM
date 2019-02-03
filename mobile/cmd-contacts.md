#Contacts类（app中特有）

###联系人类
对手机联系人的相关操作

1. <span id="Contacts.queryContacts">**Contacts.queryContacts**</span>: 获取手机通讯录信息
```
/**
 * 获取手机通讯录信息，只针对app
 * 可以根据名字和电话查询：
 *     名字的话familyName,givenName,middleName任一匹配都算匹配
 *     电话，只要有任何一个电话可以匹配到就算匹配
 * @param contact {object} 查询的条件，支持name和phone属性
 * {
 *      name: 'lori',
 *      phone: '13622334455'
 * }
 * @return {Promise} 返回的是一个promise，如果成功，promise回调参数的是联系人对象的数组
 */
queryContacts(contact)
示例：
	Contacts.queryContacts({phone:'18916628367'}).then(function(contacts) {
		if (contacts && contacts.length > 0) {
	        var num = contacts[0].phoneNumbers[0].number;
	        num = num.replaceAll('-', '').replaceAll(' ', '');
	        var verify = num === '18916628367';
	        alert(num + ":" + verify);
	    }
	}).catch(function(err) {
		console.log(err);
	});
```

1. <span id="Contacts.addContact">**Contacts.addContact**</span>: 新增联系人到手机通讯录
```
/**
 * 新增联系人到手机通讯录
 * @param contact
 * @return {Promise}
 */
addContact(contact) 
示例：
	Contacts.addContact({
		emailAddresses: [{
			label: "work",
			email: "111@example.com",
		}],
		familyName: "Nietzsche",
		givenName: "Friedrich",
	}).then(function(){
		alert('添加成功');
	}).catch(function(err){
		alert('添加失败:' + err);
	});
```

1. <span id="Contacts.updateContact">**Contacts.updateContact**</span>: 更新手机通讯录中的联系人
```
/**
 * 更新手机通讯录中的联系人
 * @param contact
 * @return {Promise}
 */
updateContact(contact) 
```

1. <span id="Contacts.deleteContact">**Contacts.deleteContact**</span>: 删除手机通讯录中的联系人
```
/**
 * 删除手机通讯录中的联系人
 * @param contact
 * @return {*}
 */
deleteContact(contact)
示例：
	Contacts.deleteContact({
	    recordID: '303'
	}).then(function(recordId){
		alert('删除成功: ' + recordId);
	}).catch(function(err){
		alert('删除失败: ' + err)
	});
```