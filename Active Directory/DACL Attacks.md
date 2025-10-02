# DACL Attacks

Trong Active Directory, các quyền DACL phổ biến bao gồm:



* **[[GenericAll - GenericWrite]]**: Cấp quyền kiểm soát hoàn toàn đối với một đối tượng (ví dụ: sửa đổi thuộc tính, đặt lại mật khẩu, v.v.).
* **[[GenericAll[[GenericAll]]- GenericWrite]]**: Cấp quyền kiểm soát hoàn toàn đối với một đối tượng (ví dụ: sửa đổi thuộc tính, đặt lại mật khẩu, v.v.).
* **[[GenericWrite]]**: Cho phép sửa đổi một số thuộc tính của đối tượng.
* **[[WriteDACL]]**: Cho phép người dùng sửa đổi chính DACL, có thể dẫn đến leo thang đặc quyền.
* **[[WriteOwner]]**: Cấp quyền sở hữu đối tượng, cho phép sửa đổi thêm đặc quyền.
* **ReadProperty**: Cho phép đọc các thuộc tính của đối tượng (ví dụ: các thuộc tính trong đối tượng người dùng).
* **AllExtendedRights**: Cấp quyền đặc biệt cho các thao tác nâng cao, như đặt lại mật khẩu hoặc kích hoạt ủy quyền.
* **Delete**: Cấp quyền xóa đối tượng.
* **ReadDACL**: Cho phép đọc quyền truy cập của đối tượng mà không thể thay đổi chúng.
* **[[ForceChangePassword]]**: Cho phép buộc người dùng thay đổi mật khẩu mà không cần biết mật khẩu hiện tại.
* **[[ReadGMSAPassword]]**: 
* **[[# ReadLAPSPassword]]**: 

<figure><img src="https://1713545152-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FZ3DvzhEDYDnE6qIbyuBL%2Fuploads%2FAILDj3hnFOdRx9kk98yX%2FDACL%20abuse%20mindmap.CnS4bNaY.png?alt=media&#x26;token=31e9298c-d271-4625-b950-cdd2aacfb9b6" alt=""><figcaption></figcaption></figure>