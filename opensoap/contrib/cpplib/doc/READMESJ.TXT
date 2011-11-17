OpenSOAP c++ library

====================
OpenSOAP��API�̈ꕔ��c++�̃N���X�Ƃ��ă��b�v�������̂ł��D
�P�D���W���[���̐����C�j���Ȃǂ�����������܂��D
�Q�DSOAP���b�Z�[�W�̐����C��͂������y�ɂȂ�܂��D

�������C���ׂĂ�API���T�|�[�g���Ă���킯�ł͂���܂���D

�t�@�C���\��

OpenSOAPBlock.h		
OpenSOAPByteArray.h	
OpenSOAPEnvelope.h	
OpenSOAPService.h	
OpenSOAPString.h	
OpenSOAPTransport.h	
OpenSOAPXMLElm.h	
������OpenSOAP API�̃��b�p�[�N���X
	
OpenSOAPInitializer.h	OpenSOAP�̏������C�I��
OpenSOAPMethod.h	SOAP���b�Z�[�W�̍쐬�C���
OpenSOAPObject.h	���ׂẴN���X�̊��N���X
OpenSOAPStructure.h	�\���̂̊��N���X

samples/
	Client.cpp	���G�ȃN���C�A���g��
	Helloc.cpp	Hello�T�[�r�X�N���C�A���g
	Hellos.cpp	Hello�T�[�r�X
	Hello.hpp	�T�[�r�X�A�N���C�A���g�C���ʎ�������

�N���X�\���F
using namespace COpenSOAP;

Object <- Block
       <- ByteArray
       <- Envelope
       <- Service
       <- String
       <- Transport
       <- XMLElm
       <- Structure(V) <- InStructure(V)
                       <- OutStructure(V)
                       <- EmptyStructure
       <- Method(V)    <- ClientMethod(T)
                       <- ServiceMethod(T)

(V)�͒��ۊ�{�N���X�C(T)�̓N���X�e���v���[�g

�ȒP�Ȏg�����F
SOAP�Ăяo�����s���ɂ́C
�܂��C���o�͍\����(Structure����h���������́j���������܂�
��F
#include "OpenSOAPMethod.h"

using namespace COpenSOAP;
using namespace std;

struct Hello_in : public Structure // �T�[�o�ɓn���l���`
{
	void CreateMessage(XMLElm& elm) const {�@
		elm.SetChildValue("name", name);
	}
	void ParseMessage(const XMLElm& elm) {   
		elm.GetChildValue("name", name);
	}

	string name; //std::string������SetChildValue�ɓn����
};

struct Hello_out : public Structure //�@�T�[�o����󂯎��l���`
{
	void CreateMessage(XMLElm& elm) const {
		elm.SetChildValue("reply", reply);
	}
	void ParseMessage(const XMLElm& elm) {
		elm.GetChildValue("reply", reply);
	}

	string reply;
};

Method�̔h���N���X����������
�T�[�r�X���i�{�f�B�[�u���b�N���j�C�\�[�v�A�N�V�����C�l�[���X�y�[�X�Ȃ�
���`���܂�
��F

class Hello_method : public Method //�K�v�ȃ��\�b�h���I�[�o�[���C�h
{
public:
	std::string GetSOAPAction()
	{ return ""; }
	std::string GetMethodName()
	{ return "Hello"; }

	std::string GetNamespaceURI()
	{ return "http://opensoap.jp/samples/hello"; }
};

ClientMethod�e���v���[�g�ŃN���C�A���g�N���X�𐶐����܂�
��F
typedef ClientMethod <Hello_in, Hello_out, Hello_method> Hello;

�G���h�|�C���g��ݒ肵�āCInvoke���\�b�h���Ăяo���܂�
��F
	Hello he;
	he.SetEndpoint("http://localhost/cgi-bin/hellos.cgi");
	try{
		he.PrintEnvelopeTo(&cout);//�G���x���[�v�̓��e���o��
		he.Invoke();
	}catch(opensoap_failed e){
		cout << "error " << hex << e.GetErrorCode() 
			<< "@"<< e.GetObjectName() << endl;
	}

�G���[���b�Z�[�W��opensoap_failed��O��ʂ��Ď󂯎��܂��D

�T�[�r�X���ł́C
ServiceMethod�e���v���[�g�ŃT�[�r�X�N���X�𐶐����āC
void ServiceMethod::ExecuteService()���������܂��D
��F
typedef ServiceMethod<Hello_in, Hello_out, Hello_method> Hello;

void Hello::ExecuteService()
{
	out.reply = "Hello! " + in.name;
}

Run���\�b�h���Ăяo���܂��D
��
	try
	{
		Hello he;
		he.Run();
	}
	catch(opensoap_failed e)
	{
		cerr << "errorcode:" << e.GetErrorCode() 
		     <<"@"<< e.GetObjectName() << endl;
	}

��Structure,XMLElm�ɂ���
 Structure����h�������\���̂́CXMLElm::SetChildValue��
���ړn�����Ƃ��ł��܂��D��������郁�b�Z�[�W��
�h������Structure�N���X�̎����Ɉˑ����܂��D
����ɁCSetChildValue�ɂ�STL��vector<T>��n�����Ƃ��ł��܂��D
���̍ۂ̃��b�Z�[�W�́C
		typename std::vector<T>::iterator it;
		for( it=ar.begin(); it<ar.end(); it++) {
			SetChildValue(name, *it);
		}
�ɂ���Đ�������܂��D

�h�������N���X�́C
CreateMessage(),ParseMessage()
����������K�v������܂��D
�N���C�A���g�݂̂��쐬����ꍇ�ŁC���݂͂̂�
�s�������ꍇ�Ȃǂ́C�K�v�ɉ�����
InStructure(create�̂ݕK�v��,parse����K�v���Ȃ��\����)
OutStructure(parse�̂ݕK�v�ŁCcreate����K�v���Ȃ��\����)
����h�����Ă��������D

�l���K�v�Ȃ��ꍇ�ibody�u���b�N�Ɏq�v�f���K�v�Ȃ��ꍇ�j��
ClientMethod�̈�����EmptyStructure���w�肵�Ă��������D
��F

struct GetDicList_out : public OutStructure
{
	void ParseMessage(const XMLElm& elm) {
		elm.GetChild("DicInfoList")
			.GetChildValue("DicInfo", DicInfoList);
	}
	vector<DicInfo> DicInfoList;
};

class GetDicList_method : public Method
{
public:
	std::string GetSOAPAction()
	{ return "http://btonic.est.co.jp/NetDic/NetDicV06/GetDicList"; }
	std::string GetMethodName()
	{ return "GetDicList"; }

	std::string GetNamespaceURI()
	{ return "http://btonic.est.co.jp/NetDic/NetDicV06"; }

};

typedef ClientMethod<EmptyStructure, GetDicList_out, 
			GetDicList_method> GetDicList;